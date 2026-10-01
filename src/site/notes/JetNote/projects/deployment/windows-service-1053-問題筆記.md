---
{"dg-publish":true,"permalink":"/jet-note/projects/deployment/windows-service-1053/","title":".NET WebApplication 佈署為 Windows Service（1053 問題筆記）","tags":["部署"],"dg-note-properties":{"title":".NET WebApplication 佈署為 Windows Service（1053 問題筆記）","tags":["部署"],"created":"2026-03-15"}}
---


# .NET WebApplication 佈署為 Windows Service（1053 問題筆記）

## 1. 問題現象

將 ASP.NET Core / WebApplication 佈署為 Windows Service 時：

Service 啟動失敗：

Error 1053
The service did not respond to the start or control request in a timely fashion

但直接執行 EXE：

InboundHandling.exe

Console 執行正常。

---

## 2. 問題根因

Windows Service 啟動時的 **Working Directory 與 Console 不同**。

### Console 啟動

Working Directory

D:\MES\MES.Background.InboundHandling

### Windows Service 啟動

Working Directory

C:\Windows\System32

如果程式中使用相對路徑，例如：

File.ReadAllText("appsettings.json")

Service 會嘗試從：

C:\Windows\System32\appsettings.json

讀取設定檔。

結果導致：

* 找不到設定檔
* 初始化失敗
* Service 無法正確完成 Start

最終出現：1053。

---

## 3. 解決方法

在 Program.cs 啟動時 **強制設定 Working Directory**。

```csharp
var builder = WebApplication.CreateBuilder(args);

Directory.SetCurrentDirectory(AppContext.BaseDirectory);

builder.Host.UseWindowsService();
```

---

## 4. 程式碼說明

### 建立 WebApplication Host

```csharp
var builder = WebApplication.CreateBuilder(args);
```

初始化：

* Configuration
* Dependency Injection
* Logging
* Kestrel

---

### 修正 Windows Service 工作目錄

```csharp
Directory.SetCurrentDirectory(AppContext.BaseDirectory);
```

將 Working Directory 指向：

EXE 所在目錄

例如：

D:\MES\MES.Background.InboundHandling

確保所有相對路徑檔案可以被正確找到。

---

### 啟用 Windows Service 模式

```csharp
builder.Host.UseWindowsService();
```

用途：

* 讓 .NET Host 與 Windows Service Control Manager 整合
* 正確回應 Service Start / Stop
* 支援 Service lifecycle

---

## 5. Console 為何正常

Console 啟動時：

Working Directory = EXE 所在目錄

Windows Service 啟動時：

Working Directory = C:\Windows\System32

因此 Console 正常但 Service 失敗。

---

## 6. 驗證方式

可加入測試程式碼：

```csharp
Console.WriteLine(AppContext.BaseDirectory);
Console.WriteLine(Directory.GetCurrentDirectory());
```

### Console

BaseDirectory = D:\MES\MES.Background.InboundHandling

CurrentDirectory = D:\MES\MES.Background.InboundHandling

### Windows Service（未修正）

BaseDirectory = D:\MES\MES.Background.InboundHandling

CurrentDirectory = C:\Windows\System32

加入 Directory.SetCurrentDirectory 後：

CurrentDirectory = D:\MES\MES.Background.InboundHandling

---

## 7. 建議的 Program.cs 標準寫法

```csharp
var builder = WebApplication.CreateBuilder(args);

// 修正 Windows Service Working Directory
Directory.SetCurrentDirectory(AppContext.BaseDirectory);

// 啟用 Windows Service
builder.Host.UseWindowsService();

var app = builder.Build();

app.Run();
```

---

## 8. Windows Service 部署目錄結構

Service 目錄應包含：

D:\MES\MES.Background.InboundHandling

* InboundHandling.exe
* InboundHandling.dll
* InboundHandling.runtimeconfig.json
* InboundHandling.deps.json
* appsettings.json

缺少 runtimeconfig 或 deps 可能導致 Service 啟動失敗。

---

## 9. 建立 Windows Service 指令

```powershell
New-Service \
-Name "MES.Background.InboundHandling" \
-BinaryPathName "D:\MES\MES.Background.InboundHandling\InboundHandling.exe" \
-DisplayName "MES.Background.InboundHandling" \
-StartupType Automatic \
-Description "MES.Background.InboundHandling"
```

---

## 10. 實務最佳實踐

所有 .NET Windows Service 建議固定加入：

```csharp
Directory.SetCurrentDirectory(AppContext.BaseDirectory);
builder.Host.UseWindowsService();
```

可避免：

* Working Directory 問題
* 設定檔讀取錯誤
* Service 1053 啟動失敗

---

## 11. Windows Service 啟動失敗常見三大原因

1. Working Directory 錯誤
2. Service Start Timeout
3. Host 未啟用 WindowsService

本案例屬於：Working Directory 問題。

---