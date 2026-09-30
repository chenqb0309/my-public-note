---
{"dg-publish":true,"permalink":"/jet-note/growing-up/c08-ai/","dg-note-properties":{}}
---


# 學習資源：AI 大模型基礎

> 課程 Area 8。這是最「雜」的一區，涵蓋從理論到工具的基礎知識。不是重點區，但每個小節都有面試價值。

---

## 提示工程與 Context Engineering

### 提示工程基礎技巧（你已經會了）
- 角色設定、清晰指令、輸出格式控制
- Few-shot 範例、Chain-of-Thought

### Context Engineering（你可能沒系統想過）
- 長上下文管理：如何把最關鍵的資訊放在 context 的開頭和結尾
- Token 預算分配：多少給系統指令、多少給使用者輸入、多少留給輸出

### 已有能力
你每天都在做這件事：Offer 角色的 system prompt 設計就是 context engineering。

---

## LLM 基本原理及 API 使用

### 基本原理
- 生成式 vs 分析式 AI
- Temperature 與 Top P 的作用（越高越隨機）
- Token 限制與上下文窗口

### API 實戰（課程案例，使用 Qwen + DashScope）
課程使用阿里雲生態，案例包含：

| CASE | 技術點 | 說明 |
|------|--------|------|
| **情感分析** | System prompt + 分類輸出 | 給一段文字，判斷正向/負向/中性 |
| **天氣 Function Calling** | Tool-use 協定 | LLM 決定呼叫天氣 API 並回傳結構化結果 |
| **表格資料萃取** | JSON Mode + Schema | 從非結構文字中提取表格欄位 |
| **智慧運維處置** | Chain-of-Thought + 工具串接 | 多步驟推理：檢測→診斷→處置建議 |

這些概念跟你設計 InboundRouter 的邏輯一致：給條件 → 做決策 → 執行行動。

### 你已經會的
- OpenAI / Anthropic / DeepSeek API 使用（比課程案例更多樣）
- Function Calling 你每天都在用（Hermes Agent 工具呼叫）
- 唯一差異：阿里雲 DashScope 生態（課程選用）vs 你用的 OpenAI 相容 API

---

## Agent 可控性與自主反思

### 幻覺控制方法（按有效性排序）
1. **Tool-use**：讓 LLM 呼叫外部 API 取得真實資料
2. **RAG**：提供檢索結果作為事實依據
3. **JSON Mode**：強制結構化輸出
4. **Human-in-the-Loop**：重要決策讓人類確認

### 思維鏈與自我反思
- CoT（Chain-of-Thought）：逐步推理
- Self-Reflection：讓 LLM 檢查自己的輸出並修正
- ReAct：Reasoning + Acting 交替進行

### 你的現狀對照
你的 Offer 角色在寫 spec 時會先規劃再執行，這就是 Plan-and-Solve 模式。
Auditor 角色做審閱，這就是 Self-Reflection 的團隊版。

---

## 多模態前沿（了解即可）

### 課程涵蓋
- MLLM（多模態大語言模型）：GPT-4o、Gemini
- 視覺感知封裝（VQA 應用）
- 影片檢索、生成方案（Sora、Kling）
- 擴散模型原理

### 對你的意義
- 如果你要做 AI 質檢（Area 3 專案），才會用到視覺模型
- 否則，**只需知道多模態是什麼**，不需要實作

---

## AI 程式設計工具

### 課程介紹的工具與實戰

| 工具 | 特點 | 課程 CASE |
|------|------|----------|
| **Cursor** | AI-first IDE，內建 Rules 系統管理開發規範 | 多張 Excel 報表合併處理、疫情即時監控大屏 |
| **Trae** | 類似 Cursor 的 AI IDE | Excel 報表處理 |
| **CodeBuddy** | VSCode 擴充，輕量 AI 輔助 | Excel 報表處理 |

### 你已經在哪
- Hermes Agent + Claude/DeepSeek — 你已經超越 IDE plugin 層級
- 你在用 Agent framework 做開發，而不只是單一檔案的自動完成
- 不過了解 Cursor/Trae 還是有價值：**面試官可能會問你用過哪些 AI 開發工具**

---

## 對你的建議

| 子項目 | 優先級 | 原因 |
|--------|--------|------|
| Context Engineering | 🟡 中 | 你已經會了，系統化一下 |
| 幻覺控制方法 | 🔴 高 | 面試必問，且你實戰用得到 |
| CoT/ReAct 原理 | 🔴 高 | 面試必問 |
| LLM 基本原理 | 🟡 中 | 你已經有實戰認知 |
| 多模態 | 🟢 低 | 了解即可 |
| AI 程式設計工具 | 🟢 低 | 你已經超越這個層級 |
