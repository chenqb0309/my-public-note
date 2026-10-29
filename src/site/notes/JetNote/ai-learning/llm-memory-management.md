---
{"dg-publish":true,"permalink":"/jet-note/ai-learning/llm-memory-management/","dg-note-properties":{}}
---


# LLM 多輪對話記憶管理

## 一句話結論

> LLM 沒有真正的記憶，記憶是靠「把歷史對話重新塞進 API」模擬出來的。

---

## messages list 結構（核心機制）

```python
messages = [
    {"role": "system",    "content": "你是助手"},        # 系統提示（永遠保留）
    {"role": "user",      "content": "我的名字是 Jet"},  # 使用者輸入
    {"role": "assistant", "content": "你好 Jet"},        # AI 回覆
    {"role": "user",      "content": "我叫什麼名字？"},   # 新一輪
    {"role": "assistant", "content": "你叫 Jet"},
]
```

每一輪對話就是 append 兩筆（user + assistant），下次 request 把整包送出去。

**為什麼不用 StringBuffer 拼接？**
- API 規格強制 messages 必須是 `array of objects`
- tool call 需要 id、參數、結果等結構化欄位，字串無法承載
- 多模態（圖片）無法用文字塞

---

## Context Window 限制

每個模型能接受的最大 token 數：

| 模型 | Context Window |
|------|---------------|
| DeepSeek V3 | 64K |
| Qwen 2.5 | 32K / 128K |
| GPT-4o | 128K |
| Claude 3.5 | 200K |

超過限制 → API 報錯或無聲截斷。

---

## 三種管理策略

### 策略一：固定窗口（最簡單，練習夠用）

```python
MAX_TURNS = 20  # 保留最近 N 輪

def add_message(messages, role, content):
    messages.append({"role": role, "content": content})
    sys_prompt = messages[0]
    recent = messages[-MAX_TURNS * 2:]  # 每輪 user + assistant
    return [sys_prompt] + recent
```

優點：實作簡單，零成本
缺點：超過 window 的直接丟掉，無法回憶

### 策略二：Token 計數裁切（進階）

```python
import tiktoken
MAX_TOKENS = 32000

def trim_messages(messages):
    total = 0
    trimmed = []
    for m in reversed(messages):       # 從最新往前數
        total += len(tiktoken.encode(m["content"]))
        trimmed.insert(0, m)
        if total > MAX_TOKENS:
            break
    return [messages[0]] + trimmed     # 確保 system prompt 永遠在
```

優點：精準控制 token 用量，不浪費空間
缺點：需要 tokenizer，每次裁切要重新計算

### 策略三：摘要壓縮（業界正規做法）

```
當前 messages 超過臨界值時，背景調用 LLM：

    1. 把早期的對話（user_1 ~ user_10）抽出來
    2. invoke LLM 做摘要：
        請將以下對話摘要成一段文字，保留關鍵事實：
        User: 我叫 Jet
        Assistant: 你好 Jet
        ...
      → "使用者名為 Jet，正在學習 LangChain"

    3. 新的 messages 變成：
        system: "你是助手。\n[早期對話摘要]：使用者名為 Jet，正在學習 LangChain"
        user_11, asst_11,  ← 保留最新的完整對話
        ...

    4. 定期在背景重複這個循環
```

優點：歷史資訊不丟失，token 控制精準
缺點：需要定期調用 LLM，有成本和延遲

---

## 三種策略選型指南

| 場景 | 推薦策略 |
|------|---------|
| 練習專案、短對話 | 固定窗口 |
| 產品上線、長對話 | token 裁切 |
| 產品上線 + 需要長期記憶 | 摘要壓縮 |

---

## invoke vs stream 實戰對照

### 各自存在的理由

| | invoke | stream |
|---|---|---|
| 回傳 | 一次拿到完整結果 | 逐塊回傳字串片段 |
| 等待時間 | 等全部跑完 | 第一塊很快出來 |
| 適合 | 背景處理、邏輯判斷、tool call | 前端即時顯示 |

### 使用場景分配

```
Agent 內部推理（每一步的判斷）  → invoke   ← 需要完整結果才能做下一步
Agent 輸出給人看              → stream   ← 人有等待焦慮
背景維護（摘要/分類/路由）     → invoke   ← 不需要人看到過程
```

### LangChain vs SK 對照

| 功能 | LangChain | Semantic Kernel |
|------|-----------|----------------|
| 一次回傳（invoke） | `llm.invoke(messages)` | `await chat.GetChatMessageContentAsync(history)` |
| 串流回傳（stream） | `llm.stream(messages)` | `await foreach (var chunk in chat.GetStreamingChatMessageContentsAsync(history))` |
| 背景並行 invoke | `asyncio.create_task(llm.ainvoke(m))` | `Task.Run(() => kernel.InvokeAsync(f))` |
| 批次並行 | `llm.abatch([m1, m2])` | 多個 create_task + Task.WhenAll |

---

## 非同步 Task 模型（C# 開發者對照）

### 建立背景任務的正確做法

```python
# Python asyncio — 正確：先 create_task 才開始跑
task1 = asyncio.create_task(llm.ainvoke(prompt1))  # ← 立即開始
task2 = asyncio.create_task(llm.ainvoke(prompt2))  # ← 也立即開始

await asyncio.sleep(0.1)      # 兩個 LLM call 同時在背景跑
result1 = await task1         # 可能已經好了，直接拿
result2 = await task2
```

```python
# Python asyncio — 錯誤：ainvoke 只是建立 coroutine，還沒開始
coro = llm.ainvoke(prompt1)   # ← 還沒跑！
result = await coro            # ← 到這裡才開始跑
```

### 批次並行

```python
# 一次送多個請求，全部回來才繼續
results = await llm.abatch([prompt1, prompt2, prompt3])
```

### C# 開發者須知

| | C# async/await | Python asyncio |
|---|---|---|
| 建立背景任務 | `Task.Run()` 或 `Task.Factory.StartNew()` | `asyncio.create_task()` |
| 底層執行 | thread pool，多 thread 平行 | 單 thread event loop 切換 |
| **應用層寫法** | `await Task.WhenAll(t1, t2)` | `await asyncio.gather(t1, t2)` |
| 應用層心智模型 | **兩者完全一致**：建立 task → await 拿結果 |

> 底層實作不同（thread pool vs event loop），但你寫應用時不需要關心這個。

---

## Ollama 的角色定位

Ollama 屬於 Model 層，不是 Agent 層：

| 層級 | 代表 | Ollama 的角色 |
|------|------|-------------|
| Chain | prompt → 解析 | Ollama 當底層模型 |
| Graph | 條件分支/狀態機 | Ollama 不參與 |
| Deep Agent | 自主規劃/工具呼叫 | Ollama 不參與 |

Ollama 只是把 OpenAI API 換成本機端點，本質仍是「prompt → text」，沒有 agent loop 或 tool orchestration。

---

## 參考

- 尚硅谷 p71-p80（Memory 章節）
- OpenAI API messages 規格文件
- `框架哲學對比-LangChain-vs-SK`（同一資料夾）
