---
{"dg-publish":true,"permalink":"/jet-note/harness/ai-processor-model/","title":"AI 應用架構框架 - Processor Model","tags":["AI-Architecture","System-Design","Mental-Model"],"dg-note-properties":{"title":"AI 應用架構框架 - Processor Model","tags":["AI-Architecture","System-Design","Mental-Model"],"created":"2026-06-26"}}
---


# AI 應用架構框架 - Processor Model

> 把 AI 從「智慧引擎」降級成「事件驅動的 Processor」—— 不改變它的能力，但改變你設計系統的方式。

相對參照：[[JetNote/harness/🔷 AI Agent 協作邊界筆記\|🔷 AI Agent 協作邊界筆記]]（控制面）、[[JetNote/harness/Agent 衍生系統設計教訓\|Agent 衍生系統設計教訓]]（實戰教訓）

---

## 一、核心抽象

### 命題

AI 暴露給外界的部分（API 端點、Prompt Template、Tool Schema）本質上是 **UI**——只是這個 UI 的 consumer 可能是人（Chat）或系統（Event）。

藏在背景的 AI 邏輯不該被認知為「智慧」或「思考」。它是 **事件驅動的 Processor**：輸入輸出都是結構化資料，跟傳統的 middleware / transformer 沒有本質區別。

```
User / System
    │ Event (trigger)
    ▼
┌─────────────────────────────┐
│ 暴露面（UI）                 │
│  - Prompt Template          │  ← Input Schema
│  - Tool Schema              │  ← API Contract
│  - Structured Output Spec   │  ← Output Schema
└─────────────────────────────┘
    │ Structured Input
    ▼
┌─────────────────────────────┐
│ Processor（AI Core）         │  ← 事件驅動的運算單元
│  - LLM call                 │     不是「思考」，是 transformation
│  - RAG retrieval            │
│  - Tool execution           │
└─────────────────────────────┘
    │ Structured Output / Action
    ▼
Database / API / Hardware
```

### 這帶來的視角轉變

| 傳統理解 | Processor Model | 架構影響 |
|---------|----------------|---------|
| LLM 是「智慧引擎」 | LLM 是 Transformer | 不會過度期待，設計更務實 |
| Prompt 是「指示」 | Prompt 是 Input Schema | 像定義 API contract 一樣定義 prompt |
| Agent 是「自主行動者」 | Agent 是 Middleware Chain | 行為可預測、可測試、可觀測 |
| AI 應用需要「理解 AI」 | AI 應用需要「設計好的 Pipeline」 | 不需要 ML 背景 |

---

## 二、Processor 的兩種暴露模式

### 2.1 Synchronous UI（人直接面對 AI）

傳統 Chat UI / 對話式介面。User 直接對 Processor 下 input。

```
User ──→ [Prompt Input] ──→ Processor ──→ [Text Output] ──→ User
```

**特性：** User 在等，latency 敏感，fallback 困難。

### 2.2 Event-Driven Processor（背景觸發）

AI 隱藏在 system boundary 內，由事件觸發，輸出送往另一個系統。

```
Sensor Event ──→ Processor ──→ DB Write / API Call / Alert
```

**特性：** User 不等，可 async，fallback 好做，observability 是關鍵。

### 2.3 兩者的關係

兩者不是互斥。同一顆 Processor 可以同時有兩種暴露面：

```
User Chat ──→ Processor ──→ User Response
                  ↑
System Event ─────┘
                  ↓
            Background Action
```

**關鍵設計決策：** 決定對外暴露多少 Processor 的行為。
- 暴露越多 → user 能直接操作 AI，但你也失去行為控制
- 暴露越少 → 系統穩定，但 user 無法彈性使用

---

## 三、與三層強制模型的對應

此模型與 [[🔷 AI Agent 協作邊界筆記#一、三層強制模型]] 的對照：

### 橫向視角（控制面 — 既有內容）

三層模型回答「怎麼控制 AI」：

| 層級 | 機制 | 可靠度 |
|------|------|--------|
| Code-level | Hook | 最高 |
| Service Boundary | MCP/CLI/API | 中 |
| Text-level | Prompt | 最低 |

### 縱向視角（架構面 — 本框架）

Processor Model 回答「AI 在系統中扮演什麼角色」：

```
                   暴露面（UI）
                       ↑
              [Input Schema / Output Schema]
                       ↑
用戶/事件 ───→ Processor ───→ 落地（DB/API/Action）
                       ↑
              [Hook / Boundary / Text]
              （三層模型提供的控制）
```

**兩者正交，疊加使用：**
- 橫向：三層模型決定 Processor 周邊的控制機制
- 縱向：Processor Model 決定 Processor 在系統架構中的位置

---

## 四、從此框架推導的設計原則

### 4.1 Processor 不該有自主權限

Processor 只能做 Input Schema 允許的事。權限由外層的 Middleware（Hook Chain / API Gateway）決定，Processor 只負責 transformation。

```
Middleware 決定「能不能做」
Processor  決定「怎麼做」
```

### 4.2 Processor 的輸入輸出必須是結構化的

Free-text 進 free-text 出的 Processor 無法測試、無法 retry、無法 monitoring。

```
✔ 好：{intent, parameters, context} → {action, payload, confidence}
✗ 壞：「幫我處理這個訂單」→「好的，已處理」
```

### 4.3 Processor 的可觀測性 === 系統的可靠性

Processor 是非確定性元件。沒有 logging / tracing 的 Processor 等於黑箱：

| 觀測點 | 監控項目 |
|--------|---------|
| Input | Schema validation pass/fail |
| Process | Latency, token count, retry count |
| Output | Structure valid, fallback triggered |
| Side effect | Tool call success/fail, DB write ok |

對應到 [[🔷 AI Agent 協作邊界筆記#四 3 靜默失敗比錯誤更危險]]：每個生命週期都要有可查詢狀態。

### 4.4 Processor 的複雜度由 Input Schema 管理

不要指望 Processor「理解意圖」。把複雜決策做在 Input Schema 的 routing 層：

```
User Input
    │
    ▼
┌─────────────────────┐
│ Router（非 LLM）      │  ← 關鍵決策在這裡，不用 LLM
│  - Keyword match     │
│  - Intent classify   │
│  - Context check     │
└─────────────────────┘
    │ routed to specific schema
    ▼
┌─────────────────────┐
│ Processor (LLM)      │  ← 只負責 transformation
│  with fixed schema   │
└─────────────────────┘
```

---

## 五、實例驗證：回顧 HM/Hook/Capsule

### Capsules（舊架構 — Processor 角色混亂）

```
Event → Capsule (Processor + Memory + Tool) → Output
         ↑─── 角色混在一起，邊界模糊
```

問題：Processor 同時負責「決策」「記憶」「工具執行」，無法個別控制。

### Hook Chain（新架構 — Processor 角色明確）

```
Event → Hook Chain (Middleware) → Processor → Structured Output
         ↑─── 控制面在這裡，不干預 transformation
```

改進：Hook 管觸發時機（Code-level），Boundary 管資源存取，Processor 只做 LLM transformation。對應三層模型，各司其職。

### 與市面上 Agent Framework 的對照

| 框架 | Processor 角色 | Middleware 層 |
|------|---------------|-------------|
| LangGraph | Node（state transformer） | Edge（conditional routing） |
| Semantic Kernel | Kernel + Plugins | Filters |
| AutoGen | Agent（message processor） | Termination / Handoff |
| **HM/Hook Chain** | **Processor（LLM call）** | **Hook / Boundary / Text** |

你在 HM 中設計的 Hook Chain 在概念上等價於這些框架的 middleware 層，差別只在於你是從 system design 角度出發，它們是從 framework API 角度。

---

## 六、職涯應用：此框架下的「LLM 應用工程師」

用 Processor Model 重新定義這個職位：

> **LLM 應用工程師 = 負責設計 Processor 的 Input Schema、Output Schema、Event Wiring 和 Observability 的人。**

需要的能力：

| 項目 | 對應此框架 | 你目前的狀態 |
|------|-----------|------------|
| Input Schema Design | Prompt Engineering + Tool Schema | 有認知（從 HM 實戰），無實作手感 |
| Output Schema Design | Structured Output + Validation | 同上 |
| Event Wiring | Agent Orchestration + Tool Calling | 認知扎實（Agent Contract 設計） |
| Observability | Logging / Tracing / Fallback | 有原則（三層日誌要求） |
| Pipeline Design | Router / Middleware / Processor 架構 | 有架構觀（Hook Chain 設計） |

**這個職業不需要手刻 Transformer 內部實作，只需要設計 Transformer 周邊的電路和通訊協定。**

就像嵌入式工程師不需要會做晶片，只需要會設計晶片周邊的電路。

---

## 七、一句話

> AI 是一座 Processor，暴露面是 UI，隱藏面是 Transformation。你的工作是設計 Input Schema 和 Output Schema，不是理解 Processor 的內部運作。
