---
{"dg-publish":true,"permalink":"/jet-note/harness/agent-vault-as-memory/","title":"Agent 記憶管理：Vault-as-Memory 模式","tags":["AI","Memory","Architecture","Vault"],"dg-note-properties":{"title":"Agent 記憶管理：Vault-as-Memory 模式","tags":["AI","Memory","Architecture","Vault"],"created":"2026-06-10","updated":"2026-06-10"}}
---


# Agent 記憶管理：Vault-as-Memory 模式

> 本文記錄 Offer 與 Jet 對於 AI Agent 記憶架構的討論共識。核心洞察：**Obsidian vault 是真實記憶層，Agent memory 只是快取索引。**

---

## 一、問題：Agent Memory 的物理限制

所有主流 Agent 平台的記憶層都有三個先天限制：

| 限制 | 描述 |
|------|------|
| **容量硬上限** | Hermes memory store ~2.2K chars，寫滿就卡 |
| **全量 injection** | 每回合所有記憶不經篩選全部塞入 context，浪費 token 且塞入噪音 |
| **供應商鎖定** | 記憶存在特定 agent 的 store，換工具就歸零 |

這不是實現問題，而是平台級架構限制。不管用哪家 OpenAI / Anthropic / Hermes，只要記憶是 session-scoped + 扁平 key-value，就會面臨同樣的困境。

---

## 二、答案：Vault-as-Memory Pattern

業界共識的最佳實踐（並非偶然，你已自然做到）：

```
Obsidian Vault（持久化、結構化、可版本控制） ← 真實記憶層
         ↕ 檔案讀寫
Agent Memory（key-value store, ~2K chars）   ← 快取索引層
```

| 做什麼 | 放哪裡 | 為什麼 |
|--------|--------|--------|
| 專案知識、技術決策、ADR | vault `program/` | 可編輯、可查、可 git 版本、工具無關 |
| 工作流程、系統約束、技能程序 | vault `meta/` + Hermes skills | 可執行、可稽核 |
| 協作範式與觀念總結 | vault `areas/` | 反思記錄、跨專案參考 |
| 使用偏好、環境路徑、輕量事實 | Agent memory (~500 chars) | 變動慢、量小、每回合都需要 |
| 當前 session 任務進度 | 不存 | 靠 conversation history |

**關鍵原則：Agent memory 只放 vault 的索引，不是 vault 的內容。**

---

## 三、現有架構對照

你目前的 setup 已經踩中這個模式：

| 你做的 | 業界對應 |
|--------|----------|
| `program/<name>/README.md` 當專案 context | Knowledge-base driven agent（Cursor / Claude Code README-driven） |
| ADR 寫在 program/ 下 | 決策記錄持久化，跨 session 可讀 |
| Skill 存在 `~/.hermes/skills/` | 程序記憶（如何做）在檔案，不在對話記憶 |
| Agent memory 只放輕量事實 | 記憶當快取，不當資料庫 |
| Obsidian vault 作為來源 | Second Brain 方法論 + 工具鏈獨立 |

---

## 四、業界記憶方案對照

| 方案 | 做法 | 適用場景 |
|------|------|----------|
| **MemGPT / Letta** | Virtual context management：main context + archival storage + reflection loop | 長時間對話、需要 swap 上下文 |
| **Mem0** | Entity extraction + importance scoring + auto-merge | 結構化記憶、多實體關聯 |
| **Generative Agents (Stanford)** | Observation → Reflection → Summary tree | 模擬人格、行為一致性 |
| **Zep** | 實體提取 + 圖譜 + 企業級 persistence | 多人協作、權限隔離 |
| **LangGraph / CrewAI** | Short-term / Long-term / Entity 三層分離 | 框架內建的多層記憶 |
| **本架構（Vault-as-Memory）** | 檔案系統當資料庫，agent memory 當索引 | 開發型 Agent、與既有知識庫整合 |

本架構與上述方案 **不衝突，可以疊加**。向量檢索仍可做為 vault content 的索引層，但主要儲存體仍在檔案系統。

---

## 五、目前 Agent Memory 該放的與不該放的

**該放的（~500 chars）：**
- 你的偏好（溝通風格、審查偏好、語言）
- 環境路徑（Obsidian vault 位置、WSL）
- 指向 vault 的索引路徑（非 vault 內容）
- 你明確說過「記得這個」的輕量事實

**不該放的（移出或合併到 vault）：**
- 專案狀態細節（production 等事實）→ program/
- 投資策略與規則 → skill + program/
- 協作哲學討論 → areas/技術筆記/
- 架構決策與約定 → meta/ + skill

---

## 六、Compaction 實作結果（2026-06-10）

第一次記憶壓縮完成，依照 Vault-as-Memory 模式清理：

| 儲存區 | 壓縮前 | 壓縮後 | 變動 |
|--------|--------|--------|------|
| memory（個人筆記） | 10 條 / 97% (2,141 chars) | 4 條 / 30% (667 chars) | 砍掉 6 條 vault 重疊內容 |
| user profile | 7 條 / 95% (1,307 chars) | 5 條 / 52% (715 chars) | 移出投資策略，合併重疊 |

**砍掉的內容類別：**
- 投資策略規則 → `program/stock-picking-strategy/` + skill 已涵蓋
- 協作哲學討論 → `areas/技術筆記/` 已記錄
- InboundHandling 生產狀態 → `program/inbound-handling/` 已涵蓋
- Vault 結構定義 → 檔案系統本身就是定義
- 個人工作反思 → `areas/技術筆記/` 更適合

**保留的內容：**
- 環境路徑與專案索引（vault 位置、repo 路徑）
- 個人背景（濃縮為一條）
- Profile 配置（auditor + stock-strategy-auditor 合併為一條）
- 溝通偏好（Kanban 串行、破壞性行動請求同意、審查偏好、誠實要求、git/WSL 環境事實）

---

## 七、後續實踐方向

1. **立即**：做一次 Agent memory compaction，砍掉與 vault 重疊的內容
2. **短期**：建立 `meta/agent-memory/` 專項，系統化定義記憶管理慣例
3. **中期**：若 Hermes 支援，評估 vector retrieval 取代全量 injection
4. **長期**：記憶管理變成自動化的 skill（定期 compaction + 一致性檢查）

---

## 七、相關文件

- `meta/agent-memory/spec.md` — Agent Memory 專項規格
- [[AI 協作範式筆記]] — Harness 協作架構背景
- [[AI 協作流程與觀念整理]] — 工作流基礎
