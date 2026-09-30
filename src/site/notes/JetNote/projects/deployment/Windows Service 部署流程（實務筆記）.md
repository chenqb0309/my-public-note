---
{"dg-publish":true,"permalink":"/jet-note/projects/deployment/windows-service/","title":"Windows Service 部署流程（實務筆記）","tags":["部署"],"dg-note-properties":{"title":"Windows Service 部署流程（實務筆記）","tags":["部署"],"created":"2026-03-15"}}
---


# Windows Service 部署流程（實務筆記）

當 .NET 應用程式需要在 Server 長期執行時，通常會部署為 Windows Service。

部署時 **必須使用系統管理員權限 PowerShell**。

原因：建立 Windows Service 需要修改系統 Service Manager。

### Step 1 開啟 Administrator PowerShell

在 Windows 搜尋：

PowerShell

右鍵 → **以系統管理員身分執行**

若沒有 Administrator 權限，建立 Service 會出現：

Access Denied

---

## Step 2 建立 Service

```powershell
New-Service \
-Name "MES.Background.InboundHandling" \
-BinaryPathName "D:\MES\MES.Background.InboundHandling\InboundHandling.exe" \
-DisplayName "MES.Background.InboundHandling" \
-StartupType Automatic \
-Description "MES.Background.InboundHandling"
```

---

## Step 3 指令語法說明

### New-Service

PowerShell 用於 **建立 Windows Service** 的指令。

會將指定 EXE 註冊到 Windows Service Manager。

---

### -Name

Service 的 **系統識別名稱**。

例如：

MES.Background.InboundHandling

用途：

* Service 控制
* 指令操作
* 系統內部識別

例如：

```powershell
Start-Service MES.Background.InboundHandling
Stop-Service MES.Background.InboundHandling
```

---

### -BinaryPathName

指定 **Service 執行的 EXE 路徑**。

例如：

D:\MES\MES.Background.InboundHandling\InboundHandling.exe

注意事項：

* 必須是完整路徑
* EXE 必須存在
* 相關 DLL / runtimeconfig 也必須在同目錄

---

### -DisplayName

Service 在 Windows 服務列表中顯示的名稱。

可與 Name 相同，也可以更易讀。

例如在 Services.msc 看到：

MES.Background.InboundHandling

---

### -StartupType

Service 啟動模式。

常見值：

Automatic

系統開機時自動啟動。

其他模式：

Manual

需要手動啟動。

Disabled

Service 被禁用。

---

### -Description

Service 的描述文字。

主要用途是方便系統管理員理解 Service 的用途。

---

## Step 4 啟動 Service

建立後可以使用：

```powershell
Start-Service MES.Background.InboundHandling
```

停止 Service：

```powershell
Stop-Service MES.Background.InboundHandling
```

重新啟動：

```powershell
Restart-Service MES.Background.InboundHandling
```

---

## Step 5 確認 Service 是否存在

```powershell
Get-Service MES.Background.InboundHandling
```

若存在會看到 Service 狀態，例如：

Running

Stopped

---

## Step 6 刪除 Service（重新部署常用）

當需要重新部署版本時，可以先刪除 Service。

```powershell
sc delete MES.Background.InboundHandling
```

刪除後再重新執行 New-Service。

---

## 13. Windows Service 部署常見問題

### 問題 1

Access Denied

原因：

PowerShell 未使用 Administrator 權限。

解法：

重新開啟 PowerShell（Run as Administrator）。

---

### 問題 2

Error 1053

原因常見為：

1 Working Directory 錯誤

2 Service 啟動初始化失敗

3 Host 未啟用 WindowsService

本筆記案例屬於：

Working Directory 問題。

---

### 問題 3

Service 啟動但立即停止

常見原因：

* EXE 發生 Exception
* 設定檔缺失
* Port 被佔用

建議先用 Console 執行 EXE 確認程式是否正常。

---

## 14. 建議的 Windows Service 部署檢查清單

部署前確認：

1 EXE 可在 Console 正常執行

2 Program.cs 包含

Directory.SetCurrentDirectory(AppContext.BaseDirectory)

builder.Host.UseWindowsService()

3 目錄包含

* exe
* dll
* runtimeconfig.json
* deps.json
* appsettings.json

4 使用 Administrator PowerShell 建立 Service

完成以上檢查後再部署 Service。

這樣大部分 Windows Service 問題都可以避免。
