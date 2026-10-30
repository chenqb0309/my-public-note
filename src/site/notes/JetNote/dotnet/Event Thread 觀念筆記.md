---
{"dg-publish":true,"permalink":"/jet-note/dotnet/event-thread/","title":"Event / Thread 觀念筆記","tags":["Concurrency","Threading","Event-Driven"],"dg-note-properties":{"title":"Event / Thread 觀念筆記","tags":["Concurrency","Threading","Event-Driven"],"created":"2025-09-09"}}
---


# Event / Thread 觀念筆記

---

## 比喻關係

* **程式 (Program)** = 老闆
* **Thread (執行緒)** = 僱員
* **Event (事件)** = 老闆分派的工作
* **Handler (事件處理器)** = 具體的任務清單

---

## 運作邏輯

1. 老闆（程式）把工作（Event）派給僱員（Thread）。
2. 僱員（Thread）依照任務清單（Handler）去執行。
3. 如果只有一個僱員，所有任務都要依序完成，沒有人能同時處理兩件事。
4. 如果有多個僱員，他們就能各自並行處理不同的工作。
5. 但當多個僱員同時操作同一份任務清單，且該任務包含共享的參數或資源，可能會發生互相覆蓋、踩到彼此工作的情況。

---

## 補充：共享資源問題

* 當多個 Thread 同時存取同一個資源時，就像多個僱員同時編輯同一份文件。
* 為了避免覆蓋或衝突，需要「協調機制」，例如：

  * **Lock / Monitor**：像是先拿到文件的使用權，其他人要等。
  * **Semaphore**：允許有限數量的人同時處理。
  * **Immutable Data**：把文件設成唯讀，每個人各拿一份副本。

---

✅ 這個比喻可以幫助你快速理解 **事件 / 執行緒 / handler / 資源共享** 的運作方式。
