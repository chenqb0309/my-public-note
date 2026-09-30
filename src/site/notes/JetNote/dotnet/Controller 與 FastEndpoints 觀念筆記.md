---
{"dg-publish":true,"permalink":"/jet-note/dotnet/controller-fast-endpoints/","title":"Controller 與 FastEndpoints 觀念筆記","tags":["ASP.NET-Core","FastEndpoints","API"],"dg-note-properties":{"title":"Controller 與 FastEndpoints 觀念筆記","tags":["ASP.NET-Core","FastEndpoints","API"],"created":"2025-06-19"}}
---


# Controller 與 FastEndpoints 觀念筆記

---

## 1. Endpoint 是什麼？

- **Endpoint** 是程式對外提供的「功能接口」或「服務入口點」。
- 具體來說，是以 URL 路徑 + HTTP 方法（GET、POST、PUT、DELETE 等）定義的唯一呼叫點。
- 客戶端就是透過 Endpoint 來呼叫你的系統功能。

---

## 2. Controller 是什麼？

- Controller 是 ASP.NET Core MVC 架構中，**一個「容器」用來管理多個 Endpoint 的集合**。
- 一個 Controller 類別通常會放多個 Action（function），每個 Action 對應一個 Endpoint。
- Controller 負責接收請求，呼叫服務，回傳結果。

---

## 3. FastEndpoints 是什麼？

- FastEndpoints 是一個專注於 Web API 的輕量級框架。
- 它把「一個 Endpoint 對應一個類別」視為基礎設計原則。
- 每個 Endpoint 類別明確實作單一功能，配合 Request/Response 型別定義，讓結構更模組化、可測試、可維護。

---

## 4. Controller 膨脹問題來源

- Controller 裡面多個 Action 互相呼叫，導致邏輯耦合。
- 商業邏輯直接寫在 Controller，缺乏清晰分層。
- Controller 集中太多責任，造成難以測試與維護。
- 缺少嚴謹的架構規範與團隊共識。

---

## 5. FastEndpoints 與 Controller 的差異

| 面向             | Controller                                    | FastEndpoints                              |
|------------------|-----------------------------------------------|--------------------------------------------|
| 類別粒度         | 一個類別管理多個 Endpoint                     | 一個類別對應一個 Endpoint                   |
| 責任劃分         | 容易混雜多職責                               | 單一職責，模組化明確                        |
| 測試便利性       | 受限於 Controller 複雜度                     | 每個 Endpoint 可獨立測試                    |
| 架構風格         | MVC 傳統架構                                 | 輕量級、類 CQRS 模式                        |
| 路由定義         | 屬性或 Minimal API                           | 類別內 Configure() 註冊                     |
| 驗證整合         | 手動或特性標註                               | 內建 FluentValidation 整合                  |

---

## 6. 使用 Controller 也可以避免問題

- 只要維持「Action 之間不互相調用」的原則。
- 商業邏輯、資料轉換、驗證等抽離到 Service 層。
- Controller 專注於接收、調用、回應。
- 嚴格遵守單一職責原則與低耦合。

---

## 7. FastEndpoints 的價值

- 幫助架構更規範，自然避免 Controller 膨脹。
- 提供強型別 Request/Response 模型，提升安全與清晰度。
- 方便測試與模組化拆分。
- 內建驗證與 Swagger 整合，提升開發效率。

---

## 8. 工程師角度建議

- 中小型專案，且團隊能嚴格控管 Controller 責任，可繼續使用 Controller 架構。
- 專案成長、需求複雜、多人協作時，建議考慮 FastEndpoints 或類似架構提升維護性。
- 持續強化 Service 層分離，避免商業邏輯堆疊在 Controller。

---

## 9. 總結

- **Endpoint 是程式對外的功能接口。**
- **Controller 是多個 Endpoint 的容器。**
- **FastEndpoints 是將每個 Endpoint 拆成獨立類別的框架，強調單一職責與模組化。**
- **Controller 膨脹的核心問題是邏輯耦合與責任不清，非 Controller 本身。**
- **嚴格分離邏輯、低耦合的 Controller 依然可用且穩健。**
- **FastEndpoints 幫助建立更清晰、可維護的 API 架構，適合中大型或成長專案。**

---