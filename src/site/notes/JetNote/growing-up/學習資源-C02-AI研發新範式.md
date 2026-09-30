---
{"dg-publish":true,"permalink":"/jet-note/growing-up/c02-ai/","dg-note-properties":{}}
---


# 學習資源：AI 研發工程師工作新範式

> 課程 Area 2。這部分**你已經在做了**，重點是把你的實戰經驗對齊到業界用語。

---

## Spec Coding 三鐵律（課程核心概念）

與你的 Offer-Corer-Taster-Auditor 直接對應：

| 課程概念 | 你的實作 | 強化方向 |
|---------|---------|---------|
| 用確定性駕馭機率性（告別感覺式編程） | Offer 寫 spec → Corer 實作 | 更嚴格的 spec 格式與驗收標準 |
| 漸進式複雜度管理 | pipeline 拆階段 | 加上 gate 機制（每階段需審閱才放行） |
| 審查-優化閉環 | Auditor 角色 | 加上量化指標（非僅人工判斷） |

### 參考資源
- **課程原文**：重新閱讀你匯出的知識圖譜 Area 2 部分
- **Spec-first development**：你已在做的模式，搜尋 "spec-driven development with AI"

---

## 從程式實現者到系統架構師

這不是技術問題，是**角色認知轉變**：

| 你現在做的 | 課程描述的「架構師」角色 |
|-----------|----------------------|
| 寫 InboundHandling 程式 | 設計整個 AMR 入站流程的 Agent 路由 |
| 跟 AI 對話產生程式碼 | 定義 AI 的任務邊界與工具集 |
| 設計 Router/Service 分層 | 設計 AI Agent 的編排層/執行層/記憶層 |

### 實作目標
下一次你在設計新系統時，先畫一張圖：
```
使用者輸入 → [意圖識別] → [任務拆解] → [分發給各 Agent] → [結果匯總]
```
而不是直接想「我要寫什麼 class」。

---

## 多智能體協作模式

課程在這區跟你實際做的 Hermes Multi-Agent pipeline 高度重疊。

### 你的現狀對照

| 課程概念 | 你的實作 |
|---------|---------|
| 主 Agent 意圖拆解、任務分發 | Offer 寫 spec，kanban 分派 |
| 子 Agent 分工與執行 | Corer（開發）、Taster（測試）、Auditor（審閱） |
| 結果回收與狀態同步 | Kanban task 狀態追蹤、block/complete 機制 |

### 你在這區只需要做的
把你的做法**文件化並用課程的語彙表達出來**，面試時就能說：「我設計了一個 multi-agent 協作系統，包含 spec 驅動開發、自動化測試驗證、第三方審計閉環。」

---

## 參考資源

- **Anthropic - Building effective agents**（再次推薦）
  https://docs.anthropic.com/en/docs/build-with-claude/agentic
- **AI Coding 新範式文章**：搜尋 "AI engineering paradigm shift 2025"
- 你已有的實戰記錄：Obsidian 中 `harness/` 目錄下的筆記本身就是參考資源
