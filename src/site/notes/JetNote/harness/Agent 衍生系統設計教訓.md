---
{"dg-publish":true,"permalink":"/jet-note/harness/agent/","dg-note-properties":{}}
---


# Agent 衍生系統設計教訓

AI agent 的 plugin / 工具鏈 / 記憶系統設計，從 HyperMemory 實戰提煉的五條跨專案原則。

---

## 1. AI 行為不可強制

**原則：** Agent 會選擇 token 成本最低的路徑，不是文件規範的路徑。

- 文件（whitepaper、skill、protocol）是**參考文獻**，不是**執行約束**
- 要強制的事寫在服務邊界（CLI gate、MCP server、API endpoint），不是 agent 的行為規範
- 如果 agent 可以直接操作儲存層，寫再多的「請先 recall 再回答」都是裝飾
- 信任邊界設計：agent 只能做你允許它做的事，不是只能看到你希望它做的事

**症狀辨識：**
- 在 skill 寫「必須先執行 A 再執行 B」→ agent 跳過 A 直接做 B
- 在 memory 放路徑資訊 → agent 繞過工具直接讀檔案
- 設計 recall 流程 → agent 從不走這個流程

**檢查：行為若不做 system 會怎樣？答案不該是「希望它做」**

---

## 2. Plugin 安裝即診斷

**原則：** Plugin lifecycle 每階段都要有明確狀態可查詢。

```
pip install → register → hook loaded → hook fired → produced effect
   pip log    ? 沒 log    ? 沒 log       ? 沒 log      ? 沒 log
```

實務上最常見的靜默失敗點：
- 安裝成功但 import 失敗 → 靜默
- import 成功但 register 失敗 → 靜默
- register 成功但 hook 從未被觸發 → 靜默
- hook 被觸發但條件不符 → 靜默

**檢查：**
- 安裝後有一行 smoke test 確認整條鏈正常
- 每個 lifecycle 階段都有 log（成功與失敗都要）
- 不依賴框架的 plugin 載入 log（框架可能沒 log）
- 提供手動驗證指令，不靠「開新 session 看效果」

---

## 3. Log 是第一優先功能

**原則：** 看不見的系統 = 無法除錯的系統 = 無法信任的系統。

- Plugin/hook 必須有健康狀態指令：哪些 hook 已註冊、上次觸發時間、成功/失敗計數
- Log 要有可追蹤 ID，能從 input → hook → query → result → effect 串起整條鏈
- 開發初期建立 Verification Loop：每次修改後有一行指令確認「它真的在工作」
- 不相信框架的 log，框架常不記錄 plugin 層的活動

**檢查：** 系統掛了還能看 log 嗎？log 儲存是否獨立於系統運作？

---

## 4. Framework 是你的最大風險因子

**原則：** Framework 文件說它能做什麼不等於它真的能這樣用。

- 假設「framework 會自動做 X」是最大風險來源（自動發現、自動載入、自動生效）
- 框架版本升級可能無聲破壞你的 plugin（改 interface、改掃描邏輯、移除 hook）
- Plugin 開發本質是逆向工程 — 除非你寫框架，否則你永遠在猜實作細節

**減輕策略：**
- 先寫 hello-world plugin 驗證整條 lifecycle，再開發正式 plugin
- 框架相關的行為發現記錄成自己的操作手冊
- Plugin 越薄越好：只做 bridge，邏輯在外部（CLI tool / library）

---

## 5. Plugin 的逆向工程本質（實作面）

- **Plugin 層越薄越好**：只做 adaptor，核心邏輯在獨立 CLI 或 library
- **每層抽象都有驗證點**：裝了？載入了？註冊了？觸發了？有效果了？
- **框架升級時 plugin 是第一波受害者**：interface 永遠是最先被改的
- **Plugin lifecycle 文件不存在就自己寫一份**：實驗得到的 lifecycle 遠比 README 準

---

## 順便記：HM 做得對的設計

不只記教訓，也記哪些決策後來證明是對的。

| 決策 | 理由 |
|------|------|
| 概念完整性優先於使用頻率 | Type 1/2/3、鏈結構、weight formula 定義了記憶如何工作，pool 長大時缺任何一塊都會出不可預測的行為 |
| Fact-Correction Invariant | 不靠人校對，靠 agent 行動後的客觀回饋（compile error、test failure）自然修正記憶；在 build/test/debug 領域最強 |
| Dual Write（即時 + 反思） | 即時寫入（對話中觸發）與事後寫入（cron job）互補，單靠任一個都會漏 |
| Cluster 歸屬跟著 prenode | 新 node 的 cluster 由 prenode 決定，不需要額外分類；Type 3 在不確定時問使用者，不猜 |
