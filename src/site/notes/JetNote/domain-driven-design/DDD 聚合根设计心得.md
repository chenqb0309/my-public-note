---
{"dg-publish":true,"permalink":"/jet-note/domain-driven-design/ddd/","title":"DDD 聚合根设计心得","tags":["Domain-Driven-Design"],"dg-note-properties":{"title":"DDD 聚合根设计心得","tags":["Domain-Driven-Design"],"created":"2026-03-04"}}
---


# DDD 聚合根设计心得

> 基于 InboundHandling 项目添加 JobId 字段的实践总结  
> 日期: 2026-03-04

---

## 💡 核心发现

### 为什么所有操作都要通过聚合根？

**关键原因：聚合根拥有最完整的上下文信息**

本次实践验证：为 `CarrierPickedEvent` 添加 `JobId` 字段时发现：

```csharp
// 数据来源分析
CarrierPickedEventArgs {
  StationId    // ← 来自 Station (聚合根)
  VehicleId    // ← 来自 Vehicle (子实体)
  JobId        // ← 来自 Vehicle.JobId (子实体属性) ⭐ 本次新增
  CarrierId    // ← 来自 Carrier (孙实体)
  Floor        // ← 方法参数
}
```

**如果在子实体触发事件会怎样？**

```csharp
// ❌ 反模式：Carrier 自己触发事件
public class Carrier 
{
  public void MarkAsPicked()
  {
    // 问题 1: JobId 在哪？需要问 Vehicle
    // 问题 2: VehicleId 在哪？需要知道我属于哪个 Vehicle
    // 问题 3: StationId 在哪？需要知道 Vehicle 在哪个 Station
    
    // 结果：被迫反向依赖父级实体，违反 DDD 原则
  }
}
```

---

## 🎯 聚合根三大职责

### 1. 完整上下文的拥有者

```csharp
public class Station  // 聚合根
{
  private Vehicle? _currentVehicle;  // 持有子实体引用
  
  public async Task<Carrier> PickCarrierAsync(int floor)
  {
    // ✅ 可以访问所有层级的信息
    var carrier = _currentVehicle.PickCarrier(floor);
    
    CarrierPicked?.Invoke(this, new CarrierPickedEventArgs
    {
      StationId = Id,                    // 自己的属性
      VehicleId = _currentVehicle.Id,    // 向下访问子实体
      JobId = _currentVehicle.JobId,     // 向下访问子实体属性
      CarrierId = carrier.Id,            // 操作返回值
      Floor = floor
    });
  }
}
```

**关键洞察：** 只有聚合根能"向下看全局"，子实体不能"向上看父级"

### 2. 不变量的守护者

```csharp
public async Task<Carrier> PickCarrierAsync(int floor)
{
  await _semaphore.WaitAsync();  // 并发控制
  try
  {
    // 验证不变量
    if (_currentVehicle == null)
      throw new InvalidOperationException($"站點 {Id} 沒有車輛");
    
    // 执行操作...
  }
  finally
  {
    _semaphore.Release();
  }
}
```

**不变量示例：**
- Station 同一时刻只能有一辆车
- 没有车时不能取载具
- 操作必须线程安全

### 3. 对外的唯一入口

```
✅ 正确调用链：
Controller → StationManager → Station.PickCarrierAsync()

❌ 禁止的访问：
Controller → Vehicle.PickCarrier()  // 跳过聚合根
Controller → Carrier.DoSomething()  // 直接访问孙实体
```

---

## 🚫 避免向上依赖的原因

### 问题 1：破坏封装

```csharp
// ❌ 反模式
public class Carrier
{
  private Vehicle _parentVehicle;  // 持有父实体引用
  
  public void DoSomething()
  {
    var jobId = _parentVehicle.JobId;  // 向上依赖
  }
}
```

**后果：**
- Carrier 需要知道 Vehicle 的存在
- Vehicle 需要知道 Station 的存在
- 形成双向/循环依赖
- 无法独立测试

### 问题 2：职责混乱

```
正确的职责分配：
- Carrier:  管理载具自身状态（ID、属性）
- Vehicle:  管理 Carriers 集合，验证 floor 有效性
- Station:  协调操作，触发事件，维护完整上下文
```

如果 Carrier 触发事件 → 需要知道 StationId、VehicleId、JobId → 职责越界

### 问题 3：事务边界混乱

```csharp
// 聚合根定义清晰的事务边界
public async Task<Carrier> PickCarrierAsync(int floor)
{
  await _semaphore.WaitAsync();  // ← 事务开始
  try
  {
    // 整个操作原子性执行
    var carrier = _currentVehicle.PickCarrier(floor);
    CarrierPicked?.Invoke(...);
    return carrier;
  }
  finally
  {
    _semaphore.Release();  // ← 事务结束
  }
}
```

如果在子实体操作 → 事务边界不清晰 → 并发问题

---

## 📐 设计模式：委派 + 协调

### 正确的协作模式

```csharp
// Vehicle: 只做业务逻辑
public Carrier PickCarrier(int floor)
{
  if (!_carriers.ContainsKey(floor))
    throw new InvalidOperationException($"第 {floor} 層沒有載具");
  
  var carrier = _carriers[floor];
  _carriers.Remove(floor);
  return carrier;  // 只返回结果，不触发事件
}

// Station: 协调 + 事件 + 完整上下文
public async Task<Carrier> PickCarrierAsync(int floor)
{
  await _semaphore.WaitAsync();
  try
  {
    // 1. 委派业务逻辑给子实体
    var carrier = _currentVehicle.PickCarrier(floor);
    
    // 2. 聚合根负责事件（因为需要完整上下文）
    CarrierPicked?.Invoke(this, new CarrierPickedEventArgs
    {
      StationId = Id,
      VehicleId = _currentVehicle.Id,
      JobId = _currentVehicle.JobId,  // ⭐ 只有这里能拿到
      CarrierId = carrier.Id,
      Floor = floor,
      PickedAt = DateTime.Now
    });
    
    return carrier;
  }
  finally
  {
    _semaphore.Release();
  }
}
```

**关键：** 业务逻辑在子实体，协调编排在聚合根

---

## 🔍 本次实践：添加 JobId 的完整链路

### 需求
为车辆到达和载具取出记录添加 `JobId`，实现 Job → Dispatch → Arrival → Pick 的完整追溯

### 修改内容

#### 1. Domain Layer - 事件参数
```csharp
// CarrierPickedEventArgs
public string? JobId { get; init; }  // ← 新增

// VehicleArrivedEventArgs  
public string? JobId { get; init; }  // ← 新增
```

#### 2. Domain Layer - 聚合根触发事件
```csharp
// Station.PickCarrierAsync
CarrierPicked?.Invoke(this, new CarrierPickedEventArgs
{
  StationId = Id,
  VehicleId = _currentVehicle.Id,
  JobId = _currentVehicle.JobId,  // ⭐ 从 Vehicle 获取
  CarrierId = carrier.Id,
  Floor = floor,
  PickedAt = DateTime.Now
});

// Station.AddVehicleAsync
VehicleArrived?.Invoke(this, new VehicleArrivedEventArgs
{
  StationId = Id,
  VehicleId = vehicleId,
  JobId = jobId,  // ⭐ 从方法参数传入
  CarrierCount = carriers.Count,
  ArrivedAt = DateTime.Now
});
```

#### 3. Infrastructure Layer - 持久化
```csharp
// CarrierPickRecord
public string? JobId { get; init; }  // ← 新增

// VehicleArrivedRecord
public string? JobId { get; init; }  // ← 新增

// MssqlStateStore - SaveCarrierPickEventAsync
INSERT INTO [dbo].[InboundCarrierPicks] 
  ([StationId], [VehicleId], [JobId], [CarrierId], [Floor], [PickedAt])
VALUES 
  (@StationId, @VehicleId, @JobId, @CarrierId, @Floor, @PickedAt)
```

#### 4. Database Schema
```sql
-- InboundCarrierPicks
ALTER TABLE [dbo].[InboundCarrierPicks]
ADD [JobId] NVARCHAR(50) NULL

CREATE NONCLUSTERED INDEX [IX_InboundCarrierPicks_JobId] 
  ON [dbo].[InboundCarrierPicks] ([JobId])

-- InboundVehicleArrivals
ALTER TABLE [dbo].[InboundVehicleArrivals]
ADD [JobId] NVARCHAR(50) NULL

CREATE NONCLUSTERED INDEX [IX_InboundVehicleArrivals_JobId] 
  ON [dbo].[InboundVehicleArrivals] ([JobId])
```

#### 5. Application Layer - 事件处理器
```csharp
// StationEventHandler.OnCarrierPicked
await _stateStore.SaveCarrierPickEventAsync(new CarrierPickRecord
{
  StationId = e.StationId,
  VehicleId = e.VehicleId,
  JobId = e.JobId,  // ⭐ 从事件参数获取
  CarrierId = e.CarrierId,
  Floor = e.Floor,
  PickedAt = e.PickedAt
});
```

### 为什么这个设计是对的？

**数据流向分析：**
```
DispatchScheduler.CreateJobAsync(jobId, ...)
  ↓
StationManager.SimulateVehicleArrival(stationId, vehicleId, jobId, ...)
  ↓
Station.AddVehicleAsync(vehicleId, carriers, jobId)
  ↓ 创建 Vehicle(vehicleId, carriers, jobId)
  ↓ Vehicle.JobId = jobId (存储在 Vehicle 中)
  ↓
Station.PickCarrierAsync(floor)
  ↓ _currentVehicle.PickCarrier(floor)  // 委派业务逻辑
  ↓ CarrierPicked 事件触发
  ↓ 事件参数包含 _currentVehicle.JobId  // ⭐ 只有 Station 能获取
  ↓
StationEventHandler.OnCarrierPicked(sender, e)
  ↓ e.JobId 持久化到数据库
```

**关键点：**
- `JobId` 存储在 `Vehicle` 实体中
- `Carrier` 不知道 `JobId` 的存在
- 只有 `Station` (聚合根) 能同时访问 `Vehicle.JobId` 和 `Carrier`
- 事件在 `Station` 触发，确保完整上下文

---

## 📊 数据完整性的价值

### 修改前：数据链断裂

```
DispatchRecord (JobId) → ❌ VehicleArrival (无 JobId) → ❌ CarrierPick (无 JobId)
```

**无法回答的问题：**
- 这个车辆是为哪个 Job 来的？
- 这个载具属于哪个 Job？
- Job 从创建到完成花了多久？
- 某个 Job 的执行过程是怎样的？

### 修改后：完整追溯链

```
DispatchRecord (JobId) → VehicleArrival (JobId) → CarrierPick (JobId)
```

**可以实现的分析：**
```sql
-- Job 端到端性能分析
SELECT 
  d.JobId,
  d.CreatedAt AS DispatchTime,
  v.ArrivedAt AS ArrivalTime,
  MIN(c.PickedAt) AS FirstPickTime,
  MAX(c.PickedAt) AS LastPickTime,
  COUNT(c.Id) AS PickCount,
  DATEDIFF(SECOND, d.CreatedAt, v.ArrivedAt) AS DispatchToArrivalSeconds,
  DATEDIFF(SECOND, v.ArrivedAt, MAX(c.PickedAt)) AS ArrivalToCompletionSeconds
FROM InboundDispatchRecords d
LEFT JOIN InboundVehicleArrivals v ON d.JobId = v.JobId
LEFT JOIN InboundCarrierPicks c ON v.JobId = c.JobId
WHERE d.CreatedAt >= DATEADD(DAY, -7, GETDATE())
GROUP BY d.JobId, d.CreatedAt, v.ArrivedAt

-- Station 按 Job 类型的负载分析
SELECT 
  StationId,
  COUNT(DISTINCT JobId) AS UniqueJobs,
  COUNT(*) AS TotalPicks,
  AVG(CAST(PicksPerJob AS FLOAT)) AS AvgPicksPerJob
FROM (
  SELECT StationId, JobId, COUNT(*) AS PicksPerJob
  FROM InboundCarrierPicks
  WHERE PickedAt >= DATEADD(HOUR, -1, GETDATE())
  GROUP BY StationId, JobId
) AS SubQuery
GROUP BY StationId
```

---

## 🎓 关键记忆点

### 口诀 1：谁拥有完整上下文？
**聚合根！**

子实体只知道自己，聚合根知道全局。

### 口诀 2：事件触发黄金法则
**事件参数需要多层级数据 → 必须在聚合根触发**

### 口诀 3：依赖方向
```
聚合根 → 子实体 → 孙实体  ✅
子实体 → 聚合根              ❌ 反模式
```

### 口诀 4：职责分离
- **子实体**：纯业务逻辑（验证、计算、状态变更）
- **聚合根**：协调编排（事件、事务、完整上下文）

### 设计检查清单

设计新功能时，问自己：

- [ ] 这个操作需要访问多个实体的数据吗？→ 放聚合根
- [ ] 这个操作需要触发包含跨层级数据的事件吗？→ 放聚合根
- [ ] 这个操作需要维护跨实体的不变量吗？→ 放聚合根
- [ ] 这个操作只修改单个实体内部状态吗？→ 可以在实体内实现
- [ ] 子实体是否需要父实体的引用？→ ❌ 重新设计

---

## 📚 项目中的聚合根实践

### InboundHandling 项目的聚合

#### Station 聚合
```
Station (Aggregate Root)
  └── Vehicle (Entity)
      └── Carrier[] (Entity/Value Object)
```

**聚合边界：**
- 一个 Station 最多一辆 Vehicle
- Vehicle 离开时整个聚合解散
- 事务边界：`_semaphore` 保护

#### DispatchScheduler 聚合
```
DispatchScheduler (Aggregate Root)
  └── DispatchJob[] (Entity)
```

**聚合边界：**
- 一个 Scheduler 管理多个 Jobs
- Job 状态变更通过 Scheduler 协调
- 事务边界：`_semaphore` 保护

**为什么 Station 和 DispatchScheduler 是不同的聚合？**
- 不同的一致性边界
- 不同的生命周期
- 不同的事务范围
- 可以独立加载和持久化

---

## 🔧 实际编码经验

### 编写聚合根方法的模板

```csharp
public async Task<TResult> DoSomethingAsync(TParam param)
{
  await _semaphore.WaitAsync();  // 1. 获取锁
  try
  {
    // 2. 验证前置条件和不变量
    if (/* 不满足条件 */)
      throw new InvalidOperationException("...");
    
    // 3. 委派业务逻辑给子实体
    var result = _childEntity.DoBusinessLogic(param);
    
    // 4. 触发领域事件（完整上下文）
    SomethingHappened?.Invoke(this, new SomethingEventArgs
    {
      AggregateRootId = Id,           // 聚合根信息
      ChildEntityId = _childEntity.Id,  // 子实体信息
      ResultData = result,            // 操作结果
      Timestamp = DateTime.Now        // 时间戳
    });
    
    // 5. 返回结果
    return result;
  }
  finally
  {
    _semaphore.Release();  // 6. 释放锁
  }
}
```

### 常见错误

#### ❌ 错误 1：在子实体触发需要上下文的事件
```csharp
public class Vehicle
{
  public event EventHandler<SomeEvent>? SomethingHappened;
  
  public Carrier PickCarrier(int floor)
  {
    var carrier = _carriers[floor];
    
    // ❌ 错误：我不知道 StationId
    SomethingHappened?.Invoke(this, new SomeEvent
    {
      StationId = ???,  // 我不知道
      VehicleId = Id    // 只知道自己
    });
  }
}
```

#### ❌ 错误 2：子实体持有父实体引用
```csharp
public class Vehicle
{
  private Station _station;  // ❌ 反向依赖
  
  public Vehicle(Station station, ...)
  {
    _station = station;
  }
}
```

#### ❌ 错误 3：外部直接访问子实体
```csharp
// Controller 中
var station = await stationManager.GetStation(stationId);
var vehicle = await station.GetCurrentVehicleAsync();
var carrier = vehicle.PickCarrier(floor);  // ❌ 跳过聚合根
```

#### ✅ 正确做法
```csharp
// Controller 中
var station = await stationManager.GetStation(stationId);
var carrier = await station.PickCarrierAsync(floor);  // ✅ 通过聚合根
```

---

## 🌟 总结

### 核心洞察

**聚合根模式的本质：**
> 在复杂的对象图中，聚合根是唯一拥有完整视角的协调者。  
> 子实体只关心自己的业务逻辑，聚合根负责编排和提供上下文。

### 本次实践的价值

通过添加 `JobId` 字段这个看似简单的需求，验证了：

1. **设计的合理性**：如果在子实体操作会遇到反向依赖问题
2. **职责的清晰性**：业务逻辑 vs 协调编排的分离
3. **数据的完整性**：只有聚合根能提供完整的事件上下文

### 设计哲学

**向下传递，向上聚合：**
- 命令向下传递（Aggregate Root → Entity）
- 数据向上聚合（Entity → Aggregate Root → Event）
- 事件在顶层触发（包含完整上下文）

**单向依赖原则：**
```
外部 → 聚合根 → 子实体 → 孙实体
     ✅       ✅        ✅
     
外部 ← 聚合根 ← 子实体 ← 孙实体
     ❌       ❌        ❌
```

---