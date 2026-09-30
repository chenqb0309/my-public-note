---
{"dg-publish":true,"permalink":"/jet-note/growing-up/c06-agent-harness/","dg-note-properties":{}}
---


# 學習資源：Agent 理論知識與 Harness Engineering

> 課程 Area 6。這是你**最有感覺**的一區，因為你每天都在做。

---

## Function Calling 與 MCP

### Function Calling
LLM 提供 API 讓模型動態選擇呼叫外部工具：
```python
# OpenAI style
tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "parameters": {...}
    }
}]
response = client.chat.completions.create(
    model="gpt-4o",
    tools=tools
)
```

### MCP（Model Context Protocol）
https://modelcontextprotocol.io/

- **MCP 是什麼**：統一的工具/資料協定，讓 LLM 可以標準化地存取外部工具和資料源
- **核心角色**：Host（你的應用）→ Client → Server（提供工具或資料）
- **與 Function Calling 的區別**：FC 是 API 格式，MCP 是完整的通訊協定（類似 USB-C 統一各種接口）

### 你的相關性
你在 Hermes Agent 中使用工具呼叫（terminal、read_file、web_search 等），本質就是 Function Calling + 工具註冊機制。MCP 就是把這件事標準化。

### 學習資源
- https://modelcontextprotocol.io/
- https://spec.modelcontextprotocol.io/

---

## Agent 自主規劃與工具開發

### 思考規劃能力的三種範式
- **反應式**：LLM 即時決定下一步（ReAct）
- **深思熟慮式**：先規劃再執行（Plan-and-Solve）
- **混合式**：規劃 + 即時調整

### 課程實作
- CoT（思維鏈）與 ReAct 實作
- Tool Use 實戰：code_interpreter、Text-to-SQL Copilot

### 你的練習目標
把你 InboundRouter 的 Capability 路由改為 Agent 決定：
```
目前：string match → 找對應 Service
未來：LLM + 當前車輛/站點狀態 → 動態決定下一步
```

---

## Agent 能力優化與效果評估

### 利用使用者反饋
- 顯式反饋（讚/踩）→ RAG 或微調
- 隱式反饋（使用者行為）→ 模式分析

### 評估方法
- **大海撈針測試**（Needle in a Haystack）：長 context 中的檢索能力
- **多跳推理評估**（Multi-hop Reasoning）
- **業務指標評估**：轉換率、處理時間、錯誤率

---

## Harness Engineering（重點）

這是課程中**跟你 Hermes 直接對應**的章節。

### 四層架構

| 層級 | 課程定義 | 你的 Hermes 對應 |
|------|---------|-----------------|
| **編排層** | 任務拆解、流程控制 | Offer 角色 → 寫 spec、開 kanban task |
| **記憶層** | 歷史、狀態、反思 | memory 工具 + 會話歷史 |
| **執行層** | 工具調用、程式執行 | terminal / write_file / read_file 等工具 |
| **反饋層** | 自動化糾錯閉環 | Auditor 審閱 + kanban block/complete |

### Harness 的關鍵機制
- **沙箱隔離**：你的 Git Worktree + workspace 隔離機制對應課程的沙箱設計
- **四層防護**：課程提到檔案系統隔離、權限粒化設計
- **工具設計原則**：最小權限、可組合性、可審計

### 學習方向
把你 Hermes Agent 的設計經驗用課程的 Harness 術語重新描述，
這就是你面試時最強的實戰案例。

---

## 參考資源

- **MCP 官方網站**：https://modelcontextprotocol.io/
- **OpenAI Function Calling Guide**：https://platform.openai.com/docs/guides/function-calling
- **Anthropic Tool Use**：https://docs.anthropic.com/en/docs/build-with-claude/tool-use
- **ReAct 論文**：https://arxiv.org/abs/2210.03629
