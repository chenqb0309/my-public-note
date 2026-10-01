---
{"dg-publish":true,"permalink":"/jet-note/ai-learning/framework-philosophy-langchain-vs-sk/","dg-note-properties":{}}
---


# LangChain vs Semantic Kernel：設計哲學對比

## 一句話結論

| | Semantic Kernel | LangChain |
|---|---|---|
| 哲學 | 先註冊後調用，一鍵完成 | 用到再當場組裝 |
| 核心物件 | Kernel（中樞） | 無中樞，裸元件組合 |
| Chain 在哪 | 封裝在 Plugin 內部 | 寫在呼叫端（explicit） |

---

## 兩個框架的關係

**Semantic Kernel 不依賴 LangChain，它們是平起平坐的競爭框架。**

- SK (.NET) → 直接調用 OpenAI SDK
- LangChain (Python) → 直接調用 openai Python SDK

沒有誰依賴誰，只是做一樣的事情。

---

## 核心差異：組織模式

### Semantic Kernel：中介者模式（Kernel 中樞）

所有服務註冊到 Kernel（類似 .NET DI Container），要什麼都透過 Kernel 拿：

```csharp
var kernel = Kernel.CreateBuilder()
    .AddOpenAIChatCompletion("gpt-4o", key)
    .Build();

// 取用都透過 kernel
var chat = kernel.GetRequiredService<IChatCompletionService>();
var result = await kernel.InvokeAsync(myFunction);
```

### LangChain：裸元件組合（無中樞）

每個元件獨立存在，用運算子串接：

```python
model = ChatOpenAI()
prompt = ChatPromptTemplate.from_messages([...])
chain = prompt | model | parser
```

---

## 學習成本 vs 靈活度

| 維度 | Semantic Kernel | LangChain |
|---|---|---|
| 上手 | 對 C# 開發者友善（DI 思維） | 對 Pythonista 友善 |
| 靈活度 | 受限 Kernel 抽象層 | 元件隨便組合 |
| Debug | 黑箱較多，要進 Kernel 內部 | 每層裸著好 trace |
| 複雜 Agent | 還在演進（2024 下半年才補） | LangGraph 成熟度高 |

---

## LangGraph 不取代 LangChain

LangGraph 建立在 LangChain 之上，不是取代關係：

- LangGraph 的節點內部依然大量使用 LC 元件（ChatModel, Tool, Runnable）
- 你無法繞過 LC 只學 LG
- 只有需要 Agent loop / Human-in-loop / 條件分支 / 跨步驟狀態共享 時，LG 才有正當理由
- 單純 RAG chain 用 LC 即可，用 LG 是自找麻煩

---

## invoke vs stream 在兩框架的對照

| 場景 | LangChain | Semantic Kernel |
|------|-----------|----------------|
| 一次回傳 | `llm.invoke(messages)` | `chat.GetChatMessageContentAsync(history)` |
| 串流回傳 | `llm.stream(messages)` | `chat.GetStreamingChatMessageContentsAsync()` |
| 背景並行 | `asyncio.create_task(llm.ainvoke(m))` | `Task.Run(() => kernel.InvokeAsync(f))` |
| 批次並行 | `llm.abatch([m1, m2])` | `await Task.WhenAll(t1, t2)` |

### 使用原則

- **Agent 內部推理**（判斷下一步、tool call）→ invoke，需要完整結果才能決定
- **輸出給人看** → stream，人有等待焦慮需要即時反饋
- **背景維護**（摘要、分類、路由）→ invoke，不需要人看到過程

### ainvoke / create_task / abatch 三者區別

```python
# 1. ainvoke — 只是建立 coroutine，還沒開始跑
coro = llm.ainvoke(prompt)
# 到 await 才真正執行

# 2. create_task — 立即在背景開始跑
task = asyncio.create_task(llm.ainvoke(prompt))
# 不需要等 await 就開始了

# 3. abatch — 批次並行，全部回來才繼續
results = await llm.abatch([p1, p2, p3])
```

### SK 註冊工具對應

| LangChain | Semantic Kernel |
|-----------|----------------|
| `agent = create_agent(llm, tools=[tool1])` | `kernel.ImportPluginFromType<MyPlugin>()` |
| 在建 agent 時傳入可用工具 | 預先註冊到 Kernel 中樞 |

都是「告訴框架有哪些工具可以用」，註冊時機不同。

---

## 可觀測性：LangSmith vs SK

LangChain 有專屬的 LangSmith 平台可以追蹤每次 LLM call、搜尋歷史、可視化 Chain 流程。SK 沒有對應產品，需要自己實作。

### SK 內建除錯方式

最簡單的做法是掛 Kernel 事件：

```csharp
kernel.FunctionInvoked += (sender, args) =>
{
    if (args.Result.Exception != null)
    {
        var funcName = args.Function.Name;
        var arguments = args.Arguments;  // AI 實際傳入的參數
        var rawPrompt = args.Result.RenderedPrompt; // 完整 prompt

        File.AppendAllText("ai_debug.log",
            $"[{DateTime.Now}] {funcName} 失敗\n" +
            $"參數：{JsonSerializer.Serialize(arguments)}\n" +
            $"錯誤：{args.Result.Exception.Message}\n---\n");
    }
};
```

這樣當 AI call 你的 function 失敗時（例如給錯欄位值），log 會記錄 AI 實際傳了什麼參數，幫助你判斷是 gateway 的 describe 不夠精確還是 AI 理解錯誤。

### 進階方案

| 需求 | 做法 |
|------|------|
| 簡單的 prompt / response log | `kernel.FunctionInvoked` 事件 |
| 結構化追蹤（分散式追蹤） | OpenTelemetry → Application Insights / Grafana Tempo |
| Chat History 視覺化 | 自己存 DB + 簡單 Web UI |

正式產品需要觀察性時補 OpenTelemetry，階段一階段二用事件 log 就夠了。

---

## 對 C# 背景開發者的意義

SK 的設計對 .NET 開發者很自然：

- Kernel = IServiceProvider（註冊服務）
- Kernel.GetRequiredService = DI 取用服務
- Kernel.InvokeAsync = Mediator 模式

LangChain 則是把 python 的 "explicit is better than implicit" 推到極致——每一步都攤在外面讓你看見。

---

## Chain / Graph / Deep Agent 三層共存

### 老師的歸納

| 層級 | 解決問題 | 代表工具 | 心智模型 |
|---|---|---|---|
| **Chain** | 快 | LCEL, Runnable | Linear pipeline，資料流過去就結束 |
| **Graph** | 穩 | LangGraph | State machine，節點+邊+條件+回圈 |
| **Deep Agent** | 複雜 | LangGraph Agent / 多 Agent 協作 | 自主規劃、反思、工具使用 |

**這是共存關係，不是版本升級。**

### 判斷原則

1. **有回圈或條件分支嗎？** → 有就用 Graph，沒有用 Chain
2. **需要跨步驟共享狀態嗎？** → 需要就用 Graph（state 是它的核心功能）
3. **流程是開放式、LLM 自主規劃？** → 是就用 Deep Agent

### 實際應用範例：AI Chat + 記憶系統

```
同一個應用中三層並存：

┌─ 使用者提問 ──────────────────────────────────┐
│                                                │
│  Graph 層（前台服務，講究穩）                    │
│  ├─ Prompt 重寫                                │
│  ├─ RAG 檢索（含條件判斷：檢索不到走不同路徑）   │
│  └─ 生成回應                                   │
│                                                │
│  Chain 層（背景固定流程，講究快）                │
│  ├─ mem0 記憶萃取（對話摘要）                    │
│  └─ 記憶回寫儲存                               │
│     （固定 pipeline，無分支、無狀態、單向）      │
│                                                │
│  Deep Agent 層（觸發式，處理開放任務）            │
│  └─ 當使用者問「幫我分析這三個月的專案進度」      │
│      進入複雜多步驟任務                          │
└────────────────────────────────────────────────┘
```

### 關鍵理解

| 流程 | 用哪層 | 為什麼 |
|---|---|---|
| mem0 背景記憶整理+回寫 | **Chain** | 資料流固定：對話→萃取→寫入，無分支 |
| 前端記憶檢索（重寫→RAG→生成） | **Graph** | 條件分支（檢索不到兜底）、多步驟狀態共享 |
| 兩者在同一應用 | **完全正常** | Graph 的節點內部也可以包 Chain |

### 一句話

> Chain 求快，Graph 求穩，Deep Agent 求多複雜都要兜住。
>
> 用最低成本解決問題的那層：能 Chain 不上 Graph，能 Graph 不上 Deep Agent。

### Ollama 在三層中的定位

Ollama 屬於 **Model 層**，不是 Agent 層：

| 層級 | Ollama 的角色 | 說明 |
|------|-------------|------|
| Chain | 當底層模型用 | 提供 prompt → text，Agent 上層做編排 |
| Graph | 不參與 | Graph 管狀態與路由，Ollama 只是其中一個 node 的模型 |
| Deep Agent | 不參與 | Agent 自主規劃、工具呼叫是上層邏輯，Ollama 只是執行推理 |

Ollama 只是把 OpenAI API 換成本機 `http://localhost:11434/v1/chat`，本質跟 `ChatOpenAI(base_url=...)` 一樣是「注入 prompt → 回吐文字」，沒有 agent loop 或 tool orchestration。

---

## 參考

- 課程脈絡：AI 應用工程師學習路徑
- 前置：`學習資源-C00-前置準備-Python速通`
- 前置：`學習資源-C01-AI框架與工具平台`
