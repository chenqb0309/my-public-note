---
{"dg-publish":true,"permalink":"/jet-note/dotnet/c-log-level-mermaid/","title":"C# Log Level 使用規範與決策圖（Mermaid 版）","tags":["C#","Logging"],"dg-note-properties":{"title":"C# Log Level 使用規範與決策圖（Mermaid 版）","tags":["C#","Logging"],"created":"2025-08-15"}}
---


# C# Log Level 使用規範與決策圖（Mermaid 版）

## 全 Log Level 決策圖

```mermaid
flowchart TD
    A[功能開始執行] --> B{需要追蹤細節以便除錯?}
    B -- 是 --> C[Trace  Verbose 記錄: 方法進入/離開, 全量參數/回傳值]
    B -- 否 --> D{是否是業務流程關鍵事件?}
    D -- 是 --> E[Information: 成功完成關鍵步驟, 狀態轉換]
    D -- 否 --> F{是否是開發/測試需檢查狀態?}
    F -- 是 --> G[Debug: 中間計算結果, 非關鍵變數]
    F -- 否 --> H[不需紀錄]

    E --> I{是否有非預期狀況但可繼續?}
    I -- 是 --> J[Warning: 缺少資料, 外部服務延遲, 可容忍異常]
    I -- 否 --> K{是否功能失敗?}
    K -- 是 --> L[Error: 無法完成功能, 需人工處理/重試]
    K -- 否 --> M[保持 Info]

    L --> N{是否致命錯誤需中斷?}
    N -- 是 --> O[Fatal: 核心模組失敗, 資料庫全斷, 必要設定缺失]
    N -- 否 --> P[保持 Error]

    C --> D
    G --> I
    J --> K
````

---

## 各 Log Level 定位與使用時機

| Level            | 用途                    | 典型場景                      |
| ---------------- | --------------------- | ------------------------- |
| Trace / Verbose  | 極細節流程追蹤，通常只在診斷或開發階段開啟 | 方法進入/離開、完整參數、全量回應內容       |
| Debug            | 開發/測試用的內部狀態檢查         | 中間計算結果、狀態旗標、非關鍵變數         |
| Information      | 正常運行的關鍵業務事件           | 成功完成重要步驟、狀態轉換、服務啟動/關閉     |
| Warning          | 非預期但可繼續的異常            | 外部 API 延遲、資料缺失但使用預設值、重試成功 |
| Error            | 功能失敗但系統可運作            | API 呼叫失敗、資料驗證錯誤、單筆交易失敗    |
| Fatal / Critical | 系統無法繼續運作，需要立即處理       | 核心服務失敗、資料庫全斷、關鍵設定缺失       |

---

## 實務應用小技巧

1. Trace / Debug 只在開發或診斷時開啟，避免生產環境 log 爆量。
2. Information 應涵蓋業務關鍵節點，方便日後分析。
3. Warning 與 Error 分級清楚，Warning 不應被誤判成重大錯誤。
4. Fatal 必須搭配全域錯誤處理器，確保能通知並安全中斷。
5. 所有 log 建議結構化，方便搜尋與分析（Serilog, NLog, ILogger 支援）。


