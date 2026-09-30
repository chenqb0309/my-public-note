---
{"dg-publish":true,"permalink":"/jet-note/harness/ai/","title":"AI 協作流程與觀念整理","tags":["AI","Collaboration","Workflow"],"dg-note-properties":{"title":"AI 協作流程與觀念整理","tags":["AI","Collaboration","Workflow"],"created":"2026-06-09","updated":"2026-06-09"}}
---


# AI 協作流程與觀念整理

## 一、核心結論

* 這套協作的本質是 Harness（駕馭工程），不是 Workflow（流程步驟）
* Workflow 規定 AI 要走的路；Harness 是防止 AI 走偏的護欄與自動導正系統
* 真正保護品質的不是 prompt 寫得多好，而是型別 + 編譯器 + 測試鏈

---

## 二、工作流三大階段

### 階段一：文件化（語義對齊）

* 人看文件（Mermaid 流程圖 + 敘述）與 AI 看規格（spec.md）分離
* 兩份文件基於同一份理解產出，不各自創作
* Corer 以 spec.md 為唯一執行依據，不讀人看文件
* 規格不清或認知不一致 → 歸責 Offer，Corer block 回報
* 單次討論放 discussions/，收斂後移 archived/

**規則：**
* spec.md 要有 Acceptance Criteria 與不做範圍
* 所有需確認的文件末尾預留 Jet 回覆區塊
* 每輪只留未決項目，已同意項移出

### 階段二：計劃 / 執行 / 審查

* 三級判斷：L1 直接做、L2 走完整 pipeline、L3 先規劃
* Offer 產 spec → 開 Kanban task → Corer 執行 → Offer 審查 → Jet 驗收
* Corer 只看 spec，嚴格 block scope 變更
* 審查獨立執行 build + test，不採信 Corer 自報結果
* 審查四維度：Build / Run / Test / Structure

**規則：**
* Test 必須在 Corer 執行前存在
* 審查要 scope 外掃描（未被 task 觸及的目錄也要檢查 namespace 殘留）
* Auditor 可作為獨立第三方參與審查

### 階段三：約束方式

* 型別約束：C# Interface + FastEndpoints 定義 API 合約，AI 不能發明簽章
* 編譯器約束：dotnet build 不過就是不過，不能被 bypass
* 測試約束：test 紅燈是 ground truth，驅動 AI 修正方向
* 審查約束：Offer 獨立執行驗證，不做 blind trust
* 文件約束：spec.md 定義 scope，超出就是 scope creep，必須 block

**原則：**
* 硬約束（編譯器、型別）比軟約束（prompt 提醒）可靠
* 約束要疊加：單一約束可能 miss，多層約束互補

---

## 三、Harness 的實際運作

```
        你 (domain 決策 + 驗收)
              │
              ▼
         Offer (spec + 審查)
         /              \
    Corer (執行)    Auditor (審計)
         \              /
         品質閘道疊加：
         型別 → 編譯器 → 測試 → 審查
```

* 每一層都是獨立的護欄
* 下層擋不住的，上層補
* 你的介入頻率 = 護欄破口的次數

---

## 四、傳統 Prompt Engineering vs 本模式

| 維度 | 傳統方式 | 本模式 |
|------|---------|--------|
| 控制手段 | 文字指令精準度 | 型別 + 編譯器 + 測試鏈 |
| 出錯反應 | 人讀 log → 改 prompt → 重跑 | 編譯器擋住 / test 紅燈自動修正 |
| 正確率來源 | LLM 機率分佈 | 編譯器確定性 + 測試覆蓋 |
| 規模化方式 | 更長 context + 更多 few-shot | 更嚴型別 + 更完整測試套件 |

---

## 五、目前敢放手的範圍 vs 還不敢的

| 敢放手的 | 還不敢的 |
|----------|---------|
| 大範圍 namespace 搬遷 | production DB schema 變更 |
| 依 spec 新增 endpoint 與 service | 跨系統同步部署 |
| 目錄結構重構 | domain 模糊、需看現場的判斷 |
| 補測試覆蓋 | |

---

## 六、已知缺口

* spec.md 仍是 markdown，不是 type-level spec（業務邏輯條件尚未被型別約束）
* 審查由 Offer 手動執行，尚未全自動
* Auditor 尚未整合進 Kanban 自動 pipeline
* 自動回饋閉環（error log → AI 自動修正）尚未實作

---

## 七、演化原則

* 不做大爆炸設計，從真實案例中 extraction
* 每完成一次 pipeline：spec 結構化 +1、審查 checklist 固定化 +1、信任感 +1
* 缺口的優先順序由實際卡關決定，不急著一次補完

---

## 案例參考

> RFIDMappingService 重組（三批搬遷，幾十個檔案改 namespace）
> 有 .NET 強型別 + 19 tests 保護，Corer 獨立完成，Offer 審查時發現 Services/ 殘留舊 namespace（6 個 build error），補修後驗證通過。

> 4D 審查獨立驗證原則的由來
> Corer 曾回報「0 error、全部 pass」，但 Offer 獨立 build 發現 6 errors（Services/ 殘留舊 namespace 未被任何 task scope 覆蓋）。從此審查強制獨立執行，不採信自報。

---

## 相關文件

* [[專案生命週期 Pipeline]] — 七階段操作流程
* `~/.hermes/skills/agent-contract/` — Offer ↔ Corer 協作合約
