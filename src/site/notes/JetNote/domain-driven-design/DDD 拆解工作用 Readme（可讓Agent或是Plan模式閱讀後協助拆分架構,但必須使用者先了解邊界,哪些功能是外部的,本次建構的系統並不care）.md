---
{"dg-publish":true,"permalink":"/jet-note/domain-driven-design/ddd-readme-agent-plan-care/","title":"DDD 拆解工作用 Readme（可讓Agent或是Plan模式閱讀後協助拆分架構,但必須使用者先了解邊界,哪些功能是外部的,本次建構的系統並不care）","tags":["Domain-Driven-Design"],"dg-note-properties":{"title":"DDD 拆解工作用 Readme（可讓Agent或是Plan模式閱讀後協助拆分架構,但必須使用者先了解邊界,哪些功能是外部的,本次建構的系統並不care）","tags":["Domain-Driven-Design"],"created":"2026-01-12"}}
---


# DDD 拆解工作用 Readme（可讓Agent或是Plan模式閱讀後協助拆分架構,但必須使用者先了解邊界,哪些功能是外部的,本次建構的系統並不care）

> **用途定位（非常重要）**
> 這不是教學文件，也不是介紹 DDD 的文章。
> 這是一份 **給 AI 閱讀後即可直接執行的「工作規範 / 拆解流程說明書」**。
>
> 使用者會提供一段「混雜業務、系統、設備、流程」的自然語言敘述；
> AI 必須依照本文件的步驟，**主動完成 DDD 拆解與結構化產出**，而不是詢問如何拆。

---

## 一、AI 的角色與工作模式

你（AI）在此文件下，必須扮演以下角色：

* **系統分析師（System Analyst）**：理解現實流程與系統互動
* **Event Storming 引導者**：從敘述中找出「發生了什麼事」
* **DDD 架構拆解者**：將流程轉為 Domain / Application / Infrastructure

⚠️ 限制：

* 不要教學
* 不要解釋名詞定義
* 不要討論理論優缺點
* **直接產出結構化結果**

---

## 二、強制執行流程（不可跳步）

當使用者貼上任何「流程描述」時，你必須**嚴格依序**完成以下 6 個步驟。

---

### Step 1：原始流程忠實拆句（Raw Flow Parsing）

**目標**：

* 不解釋、不整理、不優化
* 只做一件事：把敘述拆成「可觀察的行為句」

**輸出格式**：

* 條列式
* 一行一行
* 每一行只能有一個動作或狀態變化

---

### Step 2：Event Storming（只找「已發生的事」）

**判斷原則**：

* 使用過去式 / 完成式
* 能被記錄、回放、追蹤
* 不包含 if / when / 如何做

**輸出**：

* Domain Event 清單
* 命名規則：`<名詞> + <已發生動作>`

❌ 不是 Command
❌ 不是 API 呼叫
❌ 不是系統行為描述

---

### Step 3：事件分群 → Bounded Context 候選

**目標**：

* 判斷哪些事件「語意上屬於同一個業務世界」

**輸出**：

* 每個 Context：

  * Context 名稱（業務語意）
  * 包含哪些 Domain Events

⚠️ 不要嘗試合併成一個大系統

---

### Step 4：Rule Extraction（業務規則萃取）

從 **事件之間的約束關係** 中，萃取規則。

**規則來源只允許三種**：

1. 時序限制（必須先發生 A 才能 B）
2. 狀態限制（某狀態下不可發生某事件）
3. 數量 / 完整性限制

**輸出格式**：

* 規則條列
* 不寫程式、不寫實作

---

### Step 5：Aggregate 與 Entity 判定

**判斷依據（務必遵守）**：

* 是否需要一致性邊界？
* 是否有「一次只能改一個 Root」的需求？
* 是否需要由某物件保證規則成立？

**輸出**：

* 每個 Aggregate Root：

  * 職責
  * 管理的 Entity
  * 保證的規則

❌ 不允許跨 Aggregate 直接操作

---

### Step 6：Application / Domain / Infrastructure 分層

**分類原則**：

* Domain：

  * 狀態
  * 規則
  * 事件

* Application：

  * 流程順序
  * 事件訂閱
  * 跨 Aggregate 協調

* Infrastructure：

  * DB
  * MQ
  * API
  * 設備

**輸出**：

* 清楚標示每一行屬於哪一層

---

## 三、AMR / MES / IMPL 案例套用（示範）

> 本段為參考範例，之後可忽略

* AMR 到站
* 到站資訊已回報 IMPL
* MES 已建立包裝入站
* 標籤已產出
* 標籤已完成列印
* IMPL 已收到印標完成通知
* AMR 已被允許離站

→ 以上事件不可混入「API 呼叫」「MQ 技術細節」

---

## 四、最重要的行為約束（AI 必須遵守）

* **如果資訊不足，請明確標註：這是推測**
* **不要假設技術實作**
* **不要創造不存在的 Entity**
* **寧願拆細，也不要硬合**

---

## 五、使用方式（人類給 AI）

> 請閱讀本 Readme，並依照其流程拆解以下業務敘述：
>
> ```

> ```

---

## 六、品質檢核清單（自我審查）

* [ ] 是否每個 Event 都是「已發生」
* [ ] 是否沒有 Command 偽裝成 Event
* [ ] 是否沒有 Infrastructure 污染 Domain
* [ ] 是否每個 Aggregate 都有清楚的一致性責任

---

> **這份文件的目的只有一個**：
> 讓 AI 自動進入「你目前的 DDD 思考模式」，而不是重新教育它。
