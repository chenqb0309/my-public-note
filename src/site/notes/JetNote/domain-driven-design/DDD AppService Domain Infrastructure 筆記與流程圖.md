---
{"dg-publish":true,"permalink":"/jet-note/domain-driven-design/ddd-app-service-domain-infrastructure/","title":"DDD / AppService/ Domain / Infrastructure 筆記與流程圖","tags":["Domain-Driven-Design","Development"],"dg-note-properties":{"title":"DDD / AppService/ Domain / Infrastructure 筆記與流程圖","tags":["Domain-Driven-Design","Development"],"created":"2026-01-05"}}
---


## DDD / AppService / Domain / Infrastructure 筆記與流程圖（補充與修正版）

### 一、核心層級概念

| 層級              | 角色               | 核心責任                                     | 特性 / 注意點                                                                                      |
| --------------- | ---------------- | ---------------------------------------- | --------------------------------------------------------------------------------------------- |
| Domain / Entity | 容器 / 業務實體        | 封裝業務規則與狀態；管理 Aggregate Root 下的子 Entity   | 不直接操作 DB / 硬體；透過方法改變狀態；自我驗證狀態；保持 Aggregate 一致性                                                |
| DomainService   | 跨 Entity / 批量操作者 | 處理跨 Aggregate 或跨多 Entity 的業務規則<br>批量修改狀態 | 不存狀態，只操作 Domain Entity；提供操作入口給 AppService                                                     |
| AppService      | 流程編排者            | 組合多個 Domain 行為；決定流程順序；呼叫 Infrastructure  | 負責 orchestration；不封裝狀態規則；只做流程與外部呼叫                                                            |
| Infrastructure  | 外部世界 / 實際操作者     | DB、設備、API、檔案系統操作                         | 提供介面給 AppService 或 DomainService 呼叫；可被動(event)或主動(function)提供數據；InMemory Repository 可視為內部信息來源 |

💡 **Domain = 業務世界的模型, AppService = 導演, Infrastructure = 物理世界的手**

---

### 二、Aggregate Root 與 Entity

* **CartonEntity（Aggregate Root）**

  * 管理 BagEntity
  * 封裝拆箱規則，確保一致性
  * 方法: `CanSplit()`, `SplitBags()`

* **BagEntity**

  * 封裝拆袋規則
  * 方法: `CanBeRemoved()`, `MarkAsSplit()`

* **PrinterEntity**

  * 屬性: Queue<PrintJob>, IsBusy
  * 方法: `Enqueue()`, `Dequeue()`, `CanPrint()`

* **DomainService 範例**

  * 批量拆多個 Carton
  * 批量判斷多個 Printer 是否可列印

---

### 三、流程示意圖

```
[Controller / Frontend] --> [AppService] --> [Domain Entity / DomainService] --> [Infrastructure]

Domain Entity: CartonEntity (Aggregate Root) --> BagEntity List
DomainService: BatchCartonService / PrintJobService
Infrastructure: IPrinterHardware / IRepository / MQTTListener / APIClient

Data flow:
- Controller 發送請求
- AppService 判斷流程 -> 呼叫 Entity 方法 / DomainService
- Entity 驗證狀態並修改內部狀態
- AppService 呼叫 Infrastructure 完成外部操作 (Print / DB / API)
- Entity 狀態保持一致
```

---

### 四、C# 範例整合

```csharp
// Entity 範例
public class CartonEntity
{
    private List<BagEntity> _bags;
    public CartonEntity(List<BagEntity> bags) => _bags = bags;
    public bool CanSplit() => _bags.All(b => b.CanBeRemoved());
    public void SplitBags() { foreach(var bag in _bags) bag.MarkAsSplit(); }
}

public class BagEntity
{
    public bool IsSplit { get; private set; }
    public bool CanBeRemoved() => !IsSplit;
    public void MarkAsSplit() => IsSplit = true;
}

// AppService 範例
public class PrinterManagerService
{
    private readonly IPrinterHardware _hardware;
    private readonly List<PrinterEntity> _printers;

    public PrinterManagerService(IPrinterHardware hardware) { _hardware = hardware; _printers = new(); }

    public void SubmitJob(string printerId, PrintJobEntity job)
    {
        var printer = _printers.First(p => p.PrinterId == printerId);
        printer.Enqueue(job);
    }

    private async Task ProcessQueueAsync(PrinterEntity printer)
    {
        while(true)
        {
            if(printer.CanPrint())
            {
                printer.SetBusy(true);
                var job = printer.Dequeue();
                if(job != null) await _hardware.PrintAsync(job.Data);
                printer.SetBusy(false);
            }
            await Task.Delay(100);
        }
    }
}

public interface IPrinterHardware { Task PrintAsync(string data); }
```

---

### 五、關鍵概念補充

1. **InMemory Repository = Domain 的暫存容器**

   * 儲存多個 Entity 狀態，供 AppService 與 DomainService 使用
   * 初始化由 AppService 調用外部 snapshot / MQTT / API 取得資料，轉成 Domain Entity 再放入 InMemory Repository
2. **Domain 自行驗證狀態**

   * Entity 方法內負責可/不可操作判斷
   * Aggregate Root 確保子 Entity 一致性
3. **DomainService 處理跨 Entity / 批量操作**

   * 保持 Entity 專注單個 Aggregate
4. **Infrastructure 不干涉業務規則**

   * 只提供數據或外部操作接口
   * 可以是被動或主動來源
5. **AppService 專注流程 orchestration**

   * 不直接修改 Domain 狀態規則
   * 管理多個 Aggregate / DomainService 呼叫順序
    

