---
{"dg-publish":true,"permalink":"/jet-note/dotnet/app-settings/","title":"AppSettings 設定檔指南","tags":[".Net"],"dg-note-properties":{"title":"AppSettings 設定檔指南","tags":[".Net"],"created":"2025-03-25"}}
---


# AppSettings 設定檔指南

## 種類
AppSettings 設定檔依照環境不同，可分為以下幾種：

- `appsettings.json`（**基礎設定**，所有環境通用）
- `appsettings.Development.json`（**開發環境專用**，僅在 `Development` 模式下覆蓋 `appsettings.json`）
- `appsettings.Production.json`（**正式環境專用**，僅在 `Production` 模式下覆蓋 `appsettings.json`）
- 其他環境設定檔，例如 `appsettings.Staging.json`（**測試環境專用**）

## 設定檔載入邏輯
當應用程式運行時，會根據環境變數 `ASPNETCORE_ENVIRONMENT` 決定載入哪一個設定檔：

1. 預設會載入 `appsettings.json`。
2. 如果環境變數為 `Development`，且 `appsettings.Development.json` 存在，則覆蓋 `appsettings.json`。
3. 如果環境變數為 `Production`，且 `appsettings.Production.json` 存在，則覆蓋 `appsettings.json`。
4. 其他自訂環境，例如 `Staging`，則會嘗試載入 `appsettings.Staging.json`。

## 設定環境變數
### Windows (Command Prompt)
```sh
set ASPNETCORE_ENVIRONMENT=Development
```
### Windows (PowerShell)
```sh
$env:ASPNETCORE_ENVIRONMENT="Production"
```
### macOS / Linux
```sh
export ASPNETCORE_ENVIRONMENT=Staging
```

## `appsettings.json` 結構範例
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=myServer;Database=myDB;User Id=myUser;Password=myPass;"
  }
}
```

## `appsettings.Development.json` 覆蓋範例
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft": "Debug"
    }
  }
}
```

## 讀取 `appsettings` 設定值（C# 範例）
在 .NET Core / .NET 6+ 專案中，可以透過 `IConfiguration` 來讀取設定檔案中的值：

```csharp
using Microsoft.Extensions.Configuration;
using System;

class Program
{
    static void Main()
    {
        var builder = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
            .AddJsonFile($"appsettings.{Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT")}.json", optional: true)
            .AddEnvironmentVariables();

        IConfiguration config = builder.Build();

        string connectionString = config["ConnectionStrings:DefaultConnection"];
        Console.WriteLine($"Database Connection: {connectionString}");
    }
}
```

## 小結
- `appsettings.json` 為所有環境的基礎設定。
- `appsettings.{Environment}.json` 用於不同環境的覆蓋設定。
- `ASPNETCORE_ENVIRONMENT` 環境變數決定載入的設定檔。
- `.NET` 應用程式可透過 `IConfiguration` 讀取 `appsettings` 設定。

這樣的結構能夠確保應用程式根據不同環境載入適當的配置，提高靈活性與可維護性。