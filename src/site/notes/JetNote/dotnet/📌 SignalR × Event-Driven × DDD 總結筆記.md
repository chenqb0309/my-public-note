---
{"dg-publish":true,"permalink":"/jet-note/dotnet/signal-r-event-driven-ddd/","title":"📌 SignalR × Event-Driven × DDD 總結筆記","tags":["SignalR","Event-Driven","DDD","Real-time"],"dg-note-properties":{"title":"📌 SignalR × Event-Driven × DDD 總結筆記","tags":["SignalR","Event-Driven","DDD","Real-time"],"created":"2026-02-04"}}
---


# 📌 SignalR × Event-Driven × DDD 總結筆記

## 一、核心結論

* 在「前端畫面刷新」上：Event-Driven（SignalR 作通知層）是非常好的預設做法
* 資料真實來源仍是 API + DB

## 二、心智模型

* **SignalR** = 通知層 (Notification Layer)
* **HTTP + DB** = 真實資料層 (Source of Truth)

```
後端發事件 → 前端收到 → 再呼叫 API 更新畫面
```

## 三、前端刷新值得 Event-Driven 的原因

* **使用者體驗**：低延遲、精準通知、省流量
* **架構乾淨**：後端只發事件，前端決定畫面更新
* **可擴展性高**：新畫面只需訂閱事件即可

## 四、DDD 與 SignalR 的契合

```
DDD 思維
   ↓
Domain Event 本來存在
   ↓
EventBus / MediatR 已架好
   ↓
SignalR = 事件的其中一個 Handler
   ↓
前端即時刷新順帶完成
```

## 五、正確落地模式

### 後端 (發事件)

```csharp
await _hub.Clients.Group("Orders").SendAsync("OrderStatusChanged", orderId);
```

### 前端 (處理事件)

```js
connection.on("OrderStatusChanged", async (orderId) => {
   const latest = await fetch(`/api/orders/${orderId}`);
   updateUI(latest);
});
```

## 六、非必要 Event-Driven 的情況

* 低即時需求：每日報表、歷史紀錄
* 純本地 UI 狀態：展開/收合、分頁、篩選

## 七、風險與邊界

* 事件可能遺失，需補救機制
* 多節點部署需 Backplane (Redis)
* Debug 複雜度提升，需要良好 tracing
* Hub 不應承擔業務邏輯

## 八、事件設計原則

* 好事件命名：`OrderCreated`、`OrderApproved`、`OrderCompleted`
* 不良命名：`RefreshPage`、`ReloadTable`、`UpdateUI`
* 粒度：不太細、不太粗，前端清楚知道如何更新

## 九、檢驗清單

1. SignalR 掛掉 10 秒，重新整理資料仍正確？
2. 事件是否描述事實而非 UI 行為？
3. 新畫面能否只靠事件 + API 更新完成？

## 十、對外表述

> 在成熟的 DDD 架構下，因為 Domain Event 與 EventBus 本來就存在，SignalR 作為前端即時通知層，通常是低摩擦且自然的延伸，而不是額外負擔。
