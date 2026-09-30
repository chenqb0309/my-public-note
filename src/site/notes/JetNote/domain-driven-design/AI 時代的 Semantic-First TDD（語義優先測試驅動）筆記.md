---
{"dg-publish":true,"permalink":"/jet-note/domain-driven-design/ai-semantic-first-tdd/","title":"AI 時代的 Semantic-First TDD（語義優先測試驅動）筆記","tags":["Rule-Driven-Design"],"dg-note-properties":{"title":"AI 時代的 Semantic-First TDD（語義優先測試驅動）筆記","tags":["Rule-Driven-Design"],"created":"2026-05-14"}}
---


# AI 時代的 Semantic-First TDD（語義優先測試驅動）筆記

## 今天的重要理解

過去我以為：

```txt
TODO -> AI 生成測試 -> AI 生產 code
```

但現在我開始理解：

```txt
真正重要的不是 code 生成。
而是「語義是否被正確定義」。
```

AI 很擅長：

* 補全（Completion）
* Boilerplate（樣板程式）
* Mock setup（模擬設定）
* 測試排列（Arrange）
* Edge case（邊界案例）
* Syntax correctness（語法正確性）

但 AI 不一定真正理解：

* 系統語義（Semantic）
* 世界模型（World Model）
* Ownership（所有權）
* Invariant（不變條件）
* Aggregate Boundary（聚合邊界）
* Consistency（資料一致性）
* Event Meaning（事件意義）

因此：

```txt
不應該直接讓 AI 自由生成 unit test。
而應該由人類先定義語義，再讓 AI 補全。
```

---

# 核心理解

## 傳統 AI 使用方式

```txt
see todo and go
```

風險：

* AI 從 implementation 反推測試
* 測試只是 function 驗證
* 容易產生 semantic drift（語義漂移）
* 容易產生 implementation coupling（實作耦合）
* 命名容易崩壞
* Aggregate boundary 容易污染

---

## Semantic-First AI Workflow

應該改成：

```txt
see semantic intent and go
```

流程：

```txt
1. 人類定義語義
2. 人類定義 invariant
3. 人類定義命名
4. 人類撰寫 summary / intent
5. AI 補全測試
6. AI 生成 production code
7. 人類審查 semantic correctness
```

---

# 為什麼 Summary 很重要

因為：

```txt
測試名稱只能描述「案例」
但無法完整描述「語義」
```

例如：

```csharp
[Fact]
public void Occupied_station_cannot_accept_new_vehicle()
```

只能描述：

```txt
發生了什麼
```

但：

```csharp
/// <summary>
/// Station occupancy is exclusive.
/// A vehicle cannot belong to multiple stations simultaneously.
///
/// CheckIn represents logical occupancy acquisition,
/// not merely physical arrival detection.
/// </summary>
```

則是在描述：

* 世界模型（World Model）
* Ownership（所有權）
* Semantic Boundary（語義邊界）
* Event Meaning（事件意義）

這會大幅提升 AI 對系統的理解能力。

---

# 測試不只是驗證

過去：

```txt
測試 = 驗證 code
```

現在：

```txt
測試 = Executable Specification（可執行規格）
```

甚至：

```txt
測試 = Semantic Knowledge Base（語義知識庫）
```

未來會讀測試的：

* AI Agent
* Code Review AI
* Architecture Agent
* PR Analyzer
* Test Generator
* Refactoring Agent

因此：

```txt
測試與 summary 本身就是 AI 的語義訓練資料。
```

---

# 人類與 AI 的職責分工

## 人類負責

* Semantic（語義）
* Intent（意圖）
* Invariant（不變性）
* Boundary（邊界）
* Naming（命名）
* Failure Strategy（失敗策略）
* World Model（世界模型）
* Event Meaning（事件意義）

---

## AI 負責

* Completion（補全）
* Boilerplate（樣板）
* Mock setup（模擬設定）
* Arrange（測試排列）
* Repetitive code（重複程式）
* Edge cases（邊界案例）
* Syntax correctness（語法正確）

---

# 重要觀念

## AI 最強的不是替你思考

而是：

```txt
替你擴張已經定義好的思考。
```

因此：

```txt
AI 不應該完全自治。
而應該是 constrained autonomy（受限自治）。
```

---

# 未來的測試撰寫方式

## 不再是：

```txt
請 AI 幫我寫 unit test
```

## 而是：

```txt
我先定義語義與 intent
AI 再幫我補全測試
```

---

# Semantic Test Skeleton（語義測試骨架）

範例：

```csharp
/// <summary>
/// Vehicle occupancy is exclusive.
/// CheckIn means logical ownership acquisition.
/// </summary>
public class StationOccupancyTests
{
    [Fact]
    public void Occupied_station_cannot_accept_new_vehicle()
    {
    }
}
```

之後再交由 AI：

* 補 Arrange
* 補 Mock
* 補 Assertion
* 補 Edge Case
* 補 Builder

這樣比：

```txt
TODO -> AI 自由生成全部
```

可靠得多。

---

# 我目前的重要認知轉變

我不是要成為：

```txt
完全依賴 AI 寫 code 的人
```

而是：

```txt
定義系統語義的人
```

真正高價值的能力：

* 建模（Modeling）
* 語義設計（Semantic Design）
* Invariant 定義
* 邊界切分
* Event Meaning
* State Vocabulary
* Failure Strategy
* World Modeling

AI 則作為：

```txt
高效率語義擴張器
```

而不是架構主導者。

---

# 最後的重要提醒

如果 AI 開始自由決定：

* Aggregate Boundary
* Ownership
* Consistency Model
* Event Semantic
* Transaction Strategy

那就代表：

```txt
系統真正重要的決策權已經外包。
```

而這通常是危險的。

因此：

```txt
真正應該由人類掌握的，
不是 code control。
而是 semantic authority（語義主導權）。
```
