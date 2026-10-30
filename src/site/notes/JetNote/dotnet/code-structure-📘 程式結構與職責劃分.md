---
{"dg-publish":true,"permalink":"/jet-note/dotnet/code-structure/","title":"📘 程式結構與職責劃分","tags":[".Net"],"dg-note-properties":{"title":"📘 程式結構與職責劃分","tags":[".Net"],"created":"2025-04-21"}}
---


# 📘 程式結構與職責劃分

筆記目的是提升系統的可維護性、測試便利性，以及團隊協作效率。

本文以一個設備監控系統為例（監控含水率、處理工單、操作裝載機等），整理各層程式的功能分工。

---

## 🔹 功能層級與責任歸屬

| 層級                              | 功能說明                               | 範例檔案                                           |
| ------------------------------- | ---------------------------------- | ---------------------------------------------- |
| **BaseService**                 | 與外部系統或資料來源整合，**不包含業務邏輯**           | `OpcUaClient.cs`, `DatabaseClient.cs`          |
| **AppService**                  | 處理特定功能，整合多個 BaseService，**包含業務邏輯** | `WorkOrderService.cs`, `MoistureService.cs`    |
| **Process 編排層**                 | 執行跨功能流程，負責「流程順序與條件控制」              | `MonitoringProcess.cs`, `LoaderProcess.cs`     |
| **Worker / BackgroundTasks**    | 程式入口點，觸發流程，定時執行或事件驅動               | `MonitoringWorker.cs`, `WorkerOrchestrator.cs` |
| **應用邊界 (Application Boundary)** | 系統邏輯的進入點，處理輸入、例外與統一回應格式            | API Controller, Worker Entrypoint              |

---

## ✅ BaseService

> 功能：與外部系統或資料來源整合，**僅提供資料操作，不含判斷或流程邏輯**。

### 特性

* 單一來源資料存取
* 原子性操作
* 可被多個 AppService 重複使用

### 範例

```csharp
public interface IOpcUaClient
{
    Task<string> GetMachineWorkOrder(string equipmentId);
}

public interface IDatabaseClient
{
    Task<WorkOrder> GetOriginalWorkOrder(string workOrderId);
}
```

### ❗ Try-Catch 應視呼叫上下文決定

* **若為 private function 且有明確處理意圖，可在內部使用 try-catch。**
* **若為 public function，應避免吞錯（swallow），讓調用者依責任層級處理。**

---

## ✅ AppService

> 功能：實作特定業務功能，整合多個 BaseService 並處理邏輯。

### 特性

* 專注處理某一功能模組（如：工單、水分）
* 包含流程控制與邏輯判斷（業務邏輯, 異常處理）
* 可單元測試（透過注入 BaseService）

### Try-Catch 的角色

* 負責處理 Base 層可能拋出的錯誤，轉換為業務可理解的行為（記錄、通知、重試）
* 回傳標準化錯誤資訊，避免直接穿透系統內部例外

### 範例

```csharp
public class WorkOrderService
{
    private readonly IOpcUaClient _opcUaClient;
    private readonly IWorkOrderRepository _workOrderRepo;

    public async Task<WorkOrderInfo> GetCompleteWorkOrderInfo(string equipmentId)
    {
        try
        {
            var machineWorkOrder = await _opcUaClient.GetMachineWorkOrder(equipmentId);
            var systemWorkOrder = await _workOrderRepo.GetSystemWorkOrder(machineWorkOrder);

            if (machineWorkOrder != systemWorkOrder.Id)
            {
                // 工單不一致處理邏輯
            }

            return new WorkOrderInfo(machineWorkOrder, systemWorkOrder);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, $"取得工單資訊失敗: {equipmentId}");
            throw new ApplicationException("取得工單失敗，請稍後再試");
        }
    }
}
```

---

## ✅ Process 編排層

> 功能：實作跨模組流程的編排，例如：當水分超標 → 停止裝載機 → 發送通知。

### 特性

* 組合 AppService 執行邏輯
* 定義流程順序與條件（例如 foreach, if, try-catch）
* 聚焦「流程控制」，而非細部邏輯

### Try-Catch 的角色

* 保護流程中某一段不因單一步驟失敗而中斷整體（例如：同步一批資料中某一筆失敗）
* 可選擇略過、記錄或進行補救

### 範例

```csharp
public class LoaderProcess
{
    private readonly MoistureService _moistureService;
    private readonly LoaderService _loaderService;

    public async Task MonitorAndControlAsync()
    {
        var level = await _moistureService.GetCurrentMoisture();

        if (level > 30)
        {
            try
            {
                await _loaderService.StopLoader();
            }
            catch (Exception ex)
            {
                _logger.LogWarning(ex, "嘗試停止裝載機失敗");
                // 不中斷流程，可記錄並嘗試補救或通知
            }
        }
    }
}
```

---

## ✅ Worker / BackgroundTasks

> 功能：流程的啟動點，定時或事件驅動的任務管理。

### 特性

* 呼叫 Process 或 AppService
* 控制整體執行節奏（例如：每10秒檢查一次機台）
* 實作背景任務、排程、事件監聽等

### Try-Catch 的角色

* 作為最後防線，攔截整體流程例外，避免未處理錯誤導致應用崩潰
* 負責通知、補償處理或記錄

---

## ✅ 應用邊界（Application Boundary）

> 功能：處理來自外部系統或使用者的請求，是「系統邏輯的進入點」。

### 特性

* 接收輸入、驗證與轉換資料格式
* 集中攔截錯誤並轉為統一格式回應
* 將工作委派至 AppService 或 Process

### 錯誤集中處理

可用 Middleware 或 EntryPoint 包裹整體流程：

```csharp
public class GlobalExceptionMiddleware
{
    private readonly RequestDelegate _next;

    public async Task Invoke(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            context.Response.StatusCode = ex switch
            {
                NotFoundException => 404,
                ValidationException => 400,
                _ => 500
            };

            await context.Response.WriteAsJsonAsync(new { error = ex.Message });
        }
    }
}
```

---

## 📌 設計原則重點

| 層級           | 職責摘要                 |
| ------------ | -------------------- |
| BaseService  | 小、專一、可重用，**不含業務邏輯**  |
| AppService   | 專注處理一個功能，整合服務並加入邏輯判斷 |
| Process      | 控制多功能模組之間的流程順序與邏輯    |
| Worker       | 程式啟動點，定時執行流程或監聽事件    |
| App Boundary | 集中處理輸入驗證、錯誤攔截與格式轉換   |

> ✅ 重點不是名字叫什麼，而是「誰應該負責什麼邏輯」。

---

## 📂 建議目錄結構

```
/src
  /Infrastructure
    /External          ← BaseService (OPC UA, DB Client)
  /Application
    /WorkOrder         ← AppService
    /Moisture
  /Domain
    /Orchestration     ← Process 編排層
  /BackgroundTasks     ← Worker / 啟動主程式
  /Presentation        ← 應用邊界（API、Middleware）
```

---

> 📌 分層目的是讓每個模組「只負責它該負責的事」，讓整體系統更穩定、容易擴充與維護。
