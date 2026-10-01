---
{"dg-publish":true,"permalink":"/jet-note/domain-driven-design/ddd-stateless-singleton-memory-service/","title":"DDD 無狀態 Service 與記憶體並行架構（單一流程多向呼叫 → Singleton 併發 → Stack/Heap）","tags":["Domain-Driven-Design","Concurrency","C#","Architecture"],"dg-note-properties":{"title":"DDD 無狀態 Service 與記憶體並行架構（單一流程多向呼叫 → Singleton 併發 → Stack/Heap）","tags":["Domain-Driven-Design","Concurrency","C#","Architecture"],"created":"2026-10-01"}}
---


# DDD 無狀態 Service 與記憶體並行架構

> 原始討論：Gemini 對話（2026-09-04 建立，與 Jet 逐層推導，見文末來源）。核心結論：
> **「無狀態流程（Handler / Service）」與「有狀態實體（Aggregate / Entity）」的分離，無形中完美暗合了 CPU 指令共享與 Thread Stack 隔離的物理特性。**

---

## 一、出發點：DDD 單一流程的多向呼叫

假設業務本質只有一條主流程，但中間步驟需要呼叫多個不同的系統／服務／領域實體（多向呼叫）。關鍵在於把「核心流程控制」與「副作用／次要呼叫」解耦，有三種模式：

1. **Application Service 編排**（同步、強一致性）
   領域模型只做純粹的業務計算與狀態改變，不直接呼叫外部服務；Application Service 扮演指揮官，依序／平行呼叫其他 Port/Adapter。
   - 適用：呼叫次數少、步驟簡單、必須同步返回結果。

2. **領域事件（Domain Events）**（副作用、最終一致性）
   Aggregate Root 完成主流程狀態變更後發布領域事件（如 `OrderPlacedEvent`），其他被呼叫目標（寄信、更新庫存、寫 log）作為 Event Handler 訂閱後各自執行。
   - 優點：主流程不需要知道「後面有多少人要呼叫」，解耦且擴充性好。

3. **Saga / Process Manager**（跨服務長流程、異步 + 補償）
   引入 Saga Manager（狀態機）記錄流程走到哪一步，收到一步完成事件後觸發下一步；某步失敗則按相反順序發布補償指令。
   - 適用：分散式系統跨服務的長流程。

**核心設計原則**：領域層保持純粹（不 inject 外部服務）；區分「主業務邏輯（同步）」與「副作用（事件異步）」。

---

## 二、Singleton Service 多人呼叫為什麼不會互相卡住？

**簡短答案：不會，前提是 Service 無狀態（Stateless）。** 無狀態 Singleton 被多個 Thread 同時呼叫完全不會卡住、不需排隊。

原因在記憶體三分層模型：

| 記憶體區域 | 誰可存取 | 存放內容 | 多 Thread 會卡住嗎 |
| :--- | :--- | :--- | :--- |
| **Metaspace / Code Segment** | 所有 Thread 共用 | Singleton 方法的 CPU 指令集 | 不會（唯讀，大家一起讀） |
| **Heap（堆）** | 所有 Thread 共用 | Singleton 物件本體、成員變數 Fields | 若有人「寫」shared state 就會卡／亂 |
| **Stack（棧）** | 各 Thread 獨佔 | 傳入參數、區域變數 Local Vars | 絕對不會（各自獨立空間） |

底層運作邏輯：

- **指令是唯讀共享的**：Thread 執行的標的是一段唯讀的 CPU 指令集（在 Metaspace），不是複製整個物件。任何 Thread 都能同時讀同一段指令，如同多人同時看牆上同一張流程圖，互不干擾。
- **進度與資料隔離**：每個 Thread 的 Stack 只紀錄「自己讀到第幾行（PC 指標）」和「自己帶進來的區域變數」。Stack 不需要複製指令，只建立一個 **Frame（棧幀）**，內含 PC 指標 + 參數/區域變數 + 操作數棧。

一句話：**無狀態 Singleton 只是提供一個公共的方法指令區，CPU 為每個 Thread 配發獨立的 Stack 空間去平行執行。**

---

## 三、什麼情況下「會」卡住？（陷阱）

只有當「兩個 Thread 試圖同時修改 Heap 裡同一份共享資料」時才會衝突或需要鎖定。

```java
// ❌ 陷阱 A：共享可變成員變數
public class OrderService {
    private String currentUserId; // 共享變數！Thread A 寫入後可能被 Thread B 蓋掉
    public void processOrder(OrderRequest req) {
        this.currentUserId = req.getUserId();
        // 若此時 Thread B 進來改掉，Thread A 後續處理就拿錯人
    }
}
```

```java
// ❌ 陷阱 B：加鎖強制排隊
public synchronized void processOrder(OrderRequest req) { ... }
// synchronized / Lock 會讓所有請求強行排隊
```

真正的效能瓶頸通常在 Service 之外：

- **資料庫鎖（DB Lock）**：多 Thread 更新同一筆資料的 Row-level Lock，後到者等待。
- **連線池耗盡（Connection Pool Exhaustion）**：DB 或 HTTP Client 連線池被佔滿。
- **阻塞式 I/O（Blocking I/O）**：第三方 API 回應極慢，佔用 Thread。

---

## 四、黃金法則：設計完全無狀態的 Service

```java
@Service // Singleton
public class OrderApplicationService {
    // 依賴的 Repository / Adapter 是唯讀、無狀態的，可安全共享
    private final OrderRepository orderRepository;
    private final PaymentAdapter paymentAdapter;

    public void processOrder(OrderRequest req) {
        // 1. 區域變數放在 Thread Stack，每個 Request 獨立
        Order order = orderRepository.findById(req.getOrderId());
        // 2. 業務邏輯與狀態改變發生在 Entity 內部，非 Service 內部
        order.changeStatus(OrderStatus.PROCESSING);
        // 3. 呼叫外部 Adapter
        paymentAdapter.pay(order.getAmount());
        orderRepository.save(order);
    }
}
```

要點：**不要在 Service 定義可變的成員變數；一切資料透過方法參數傳遞；所有狀態變更交由領域物件或區域變數處理。**

---

## 五、核心突破：架構與記憶體物理特性的完美對映

這是整篇的領悟點 —— **高層次的架構思想與底層硬體運作邏輯在終點交會。**

| DDD / OOP 設計 | 記憶體物理特性 | 為何這樣設計效能最高 |
| :--- | :--- | :--- |
| Command Handler / Application Service（無狀態流程，Singleton） | Metaspace / Code Segment（唯讀指令區） | 指令載入一次，所有 Core/Thread 並行讀取，不需複製也不需加鎖 |
| 局部變數 / 方法參數（執行過程傳遞資料） | Thread Stack（Thread 專屬獨立棧） | 存取最快，Thread 獨佔、完全不需要同步（No Lock） |
| Aggregate / Entity（有狀態核心領域物件） | Heap Space（動態分配堆） | 短生命週期物件，處理完沒有 Singleton 強引用，迅速被 Young Generation GC 回收，不洩漏 |

**為什麼順應記憶體特性可以避免災難：**

- 不破壞 CPU 並行能力：若把狀態寫進 Singleton，為保護 Heap 共享狀態勢必加 `synchronized`/`Lock`，把多核心並行直接降級成單排順序執行。
- 避免 GC 災難：若 Singleton 意外長久持有使用者資料物件的引用，物件會從 Young Gen 跑到 Old Gen，最終引發 Full GC 全系統停頓。
- 避免記憶體污染：多 Thread 同時寫 Heap 共享資料，會讓 Stack 邏輯拿到被覆蓋的髒資料（Race Condition）。

---

## 六、腦內模型（比喻）

- **Method Area（程式碼）＝ 牆上的流程圖**：所有人同時看，不擠、不卡。
- **Heap（堆）＝ 公用大桌子**：Singleton 物件放這。若桌面有共用筆記本（成員變數）：只看（唯讀）不卡；兩人同時拿筆改它 → Race Condition，加鎖 → 排隊卡住。
- **Stack（棧）＝ 每人口袋的小筆記本**：各自紀錄自己的參數與中間變數，彼此完全隔絕。

補充一個關鍵修正：**Thread 不是複製 Code Instruction**（指令很大，每個 Thread 複製一份記憶體會爆），而是複製「執行的位置與狀態」（PC 指標 + 資料）。指令只有一份，存在 Method Area，大家共用同一份。

---

## 交集延伸：職責分工總結

| 角色 | 狀態屬性 | 生命週期 | 在 Thread 裡的定位 |
| :--- | :--- | :--- | :--- |
| CommandHandler / Service（流程） | 無狀態 | Singleton（啟動即存在，只有一份） | 共享指令集，所有 Thread 同時讀取執行，不占 Heap 狀態 |
| Aggregate / Entity（儲存的物件） | 有狀態 | 短暫（用完即丟或靠 DB 持久化） | Thread 私有／區域變數，載入到自己的 Stack 處理，絕不跨 Thread 共享 |

**「流程（Handler）保持無狀態，才能放心讓多個 Thread 同時跑；有狀態的物件（Entity）只在流程執行期間短暫載入、局部處理。」** 這設計不只讓系統在高併發下不會互鎖，也讓單一職責（SRP）變得分明。

---

## 來源

- Gemini 對話（建立於 2026-09-04），標題即「DDD 單一流程多向呼叫解法」
- 原始短連結：https://share.gemini.google/S0irt6ZhIzly
- 收錄於本 vault 的短連結：https://share.gemini.google/cCyparrtl5nK

## 延伸閱讀

- [[JetNote/dotnet/執行緒、資源、控制權、Task 的完整整理\|執行緒、資源、控制權、Task 的完整整理]]
- [[JetNote/domain-driven-design/DDD AppService Domain Infrastructure 筆記與流程圖\|DDD AppService Domain Infrastructure 筆記與流程圖]]
- [[JetNote/dotnet/📌 SignalR × Event-Driven × DDD 總結筆記\|📌 SignalR × Event-Driven × DDD 總結筆記]]