---
{"dg-publish":true,"permalink":"/jet-note/domain-driven-design/event-storming-rule-extraction/","title":"Event Storming + 業務規則歸納（Rule Extraction）方法流程","tags":["Domain-Driven-Design"],"dg-note-properties":{"title":"Event Storming + 業務規則歸納（Rule Extraction）方法流程","tags":["Domain-Driven-Design"],"created":"2026-01-09"}}
---


# Event Storming + 業務規則歸納（Rule Extraction）方法流程

> 本文件整理一套**可重複使用的分析方法**，用來從混雜的業務／系統流程中，**長出 Domain、Aggregate 與 Service 邊界**。
> 並以「江氏標籤轉換暨後續操作」案例作為實例輔助說明。

---

## 一、方法總覽（先給全貌）

這不是畫圖技巧，而是一種**思考順序**：

1. **事件優先（Event First）**：先找「發生了什麼事」
2. **規則浮現（Rule Extraction）**：哪些事不能亂做、不能只做一半
3. **一致性邊界（Aggregate Boundary）**：誰必須一起被正確處理
4. **責任歸屬（Who decides）**：誰有權判斷「可不可以」
5. **流程與世界分離**：內部決策 vs 外部行為

> 關鍵原則：**不是先想 class，而是讓 class 被逼出來**

---

## 二、Step 1：事件攤平（Event Storming）

### 做法

* 用「**已經發生的事**」來描述
* 不出現 if / loop / API / DB
* 動詞使用過去式

### 案例對應

從原始描述中，我們可以抽出這些事件：

* 包裝履歷已存在於江氏系統
* 製令被要求轉入 MES
* 外箱條碼被要求用於出貨或儲位
* 內袋發生拆裝數量
* 產品 Rev 資訊變更
* 標籤被更新

📌 到這一步 **完全不談系統怎麼做**

---

## 三、Step 2：標記「不能只做一半」的地方（Rule Extraction）

### 問的不是「怎麼做」而是：

> **什麼情況下，如果只改一部分，業務會出錯？**

### 案例中的規則浮現

#### 規則 A（外箱需求）

* 只因物流 / 儲位需求
* 內袋追溯不可受影響

➡ 規則：**只能換外箱，不能動內袋**

---

#### 規則 B（拆裝出庫）

* 只有被拆的內袋才失效

➡ 規則：**只換異動內袋，其餘保持**

---

#### 規則 C（Rev 變更）

* 原本識別整體失效

➡ 規則：**全部標籤必須一致更新**

📌 注意：這些不是流程，而是**不可違反的業務約束**

---

## 四、Step 3：找一致性邊界（Aggregate Boundary）

### 關鍵提問

> **哪些東西必須一起被正確處理，否則業務語意會壞掉？**

### 案例判斷

* 外箱 + 多個內袋
* 有時一起動，有時部分動
* 規則是「整體視角」決定

➡ 自然長出：

```text
Package（Aggregate Root）
 ├─ OuterBoxLabel
 └─ BagLabel (1..n)
```

📌 Bag 不自己決定命運，它的變化由 Package 判斷

---

## 五、Step 4：責任歸屬（Who decides what）

### 判斷原則

* **誰能說「可不可以」 → Domain / Entity**
* **誰只是照規則執行 → Service**

### 案例套用

* 「這種情況能不能只換外箱？」 → Package
* 「這次是 Rev 變更嗎？」 → Package
* 「依製令逐批處理轉換」 → DomainService

➡ 分工自然形成：

```text
Package
 - CanUpdateOuterLabel()
 - SplitBags(changedBags)
 - ReviseAll()

LabelConversionService
 - HandleByManufacturingOrder()
```

---

## 六、Step 5：流程與現實世界分離

### 核心切割線

| 問題類型  | 所屬層            | 說明               |
| ----- | -------------- | ---------------- |
| 能不能換  | Domain         | 規則、狀態、一致性        |
| 什麼時候換 | AppService     | 流程、批次、順序         |
| 怎麼印   | Infrastructure | MES / 印表機 / RFID |

### 案例映射

* Domain：決定哪些 Label 需要更新
* AppService：依製令逐筆觸發
* Infrastructure：實際呼叫 MES 印標

📌 Domain 永遠不直接「印」

---

## 七、方法總結（可重複使用）

### 一句話版本

> **先讓事件說話，再讓規則現形，最後讓物件被逼出來**

### 你這次實際做的事情

* 用反問法逼出「不能只做一半」
* 讓一致性邊界自然浮現
* 沒急著畫 class
* 沒被系統 API 帶著跑

這正是成熟 DDD 分析的樣子。

---

## 八、判斷你有沒有用對方法（自我檢核）

✔ Domain 規則可以獨立閱讀，不看流程
✔ 改需求時，只動 Domain 規則，不重寫流程
✔ 外部系統全壞，Domain 邏輯仍成立

如果三個都成立，代表這套方法是成功的。
