---
{"dg-publish":true,"permalink":"/jet-note/projects/radzen-blazor-server/","title":"📘 Radzen + Blazor Server 前端樞構筆記（最小可用框架）","tags":["Blazor","Radzen","UI","ASP.NET-Core"],"dg-note-properties":{"title":"📘 Radzen + Blazor Server 前端樞構筆記（最小可用框架）","tags":["Blazor","Radzen","UI","ASP.NET-Core"],"created":"2025-08-06"}}
---


# 📘 Radzen + Blazor Server 前端樞構筆記（最小可用框架）

---

## I. 專案初始化與套件安裝

### 1. 建立 Blazor Server 專案

```bash
dotnet new blazorserver -n RadzenStarter
cd RadzenStarter
```

### 2. 安裝 Radzen 元件套件

```bash
dotnet add package Radzen.Blazor
```

---

## II. 全域元件與服務註冊設定

### 3. 修改 `_Imports.razor` 加入命名空間

```razor
@using Radzen
@using Radzen.Blazor
```

### 4. 設定 `Program.cs` 注入 Radzen 服務

```csharp
builder.Services.AddScoped<DialogService>();
builder.Services.AddScoped<NotificationService>();
builder.Services.AddScoped<TooltipService>();
builder.Services.AddScoped<ContextMenuService>();
```

---

## III. 樣式與腳本整合（UI/互動支援）

### 5. 修改 `Host.cshtml` 引入 CSS / JS

```html
<!-- Radzen 樣式 放置在head區域 -->
<link rel="stylesheet" href="_content/Radzen.Blazor/css/material-base.css">
<link rel="stylesheet" href="_content/Radzen.Blazor/css/standard-base.css">

<!-- Radzen JS 放置在body區域-->
<script src="_content/Radzen.Blazor/Radzen.Blazor.js"></script>
```

---

## IV. Layout 與元件配置（通知、對話框等）

### 6. `MainLayout.razor` 配置核心元件

```razor
@inherits LayoutComponentBase
<RadzenLayout>
    <RadzenBody>
        @Body
    </RadzenBody>
    <RadzenComponents @rendermode="InteractiveServer" />
</RadzenLayout>
```

---

## V. 實作驗證：測試元件運作

### 7. 新增 `Pages/Test.razor`

```razor
@page "/test"
@inject NotificationService NotificationService
@inject DialogService DialogService

<h3>Radzen 測試頁面</h3>

<RadzenButton Text="顯示通知"
              Click="@ShowNotification"
              Style="margin-bottom:10px" />

<RadzenButton Text="顯示對話框"
              Click="@ShowDialog" />

@code {
    void ShowNotification()
    {
        NotificationService.Notify(new NotificationMessage
        {
            Severity = NotificationSeverity.Success,
            Summary = "成功",
            Detail = "這是一則通知訊息",
            Duration = 4000
        });
    }

    async Task ShowDialog()
    {
        await DialogService.OpenAsync("對話框",
            ds => @<div>這是對話框的內容</div>,
            new DialogOptions() { Width = "400px", Height = "200px" });
    }
}
```

### 8. 新增導覽連結 `Shared/NavMenu.razor`

```razor
<NavLink href="test" Match="NavLinkMatch.All">
    <span class="oi oi-flash" aria-hidden="true"></span> 測試頁面
</NavLink>
```

---

## VI. 進階備忘

### 9. Radzen 主題選擇

| 樣式名稱        | CSS 檔名                                         |
| ----------- | ---------------------------------------------- |
| Material 主題 | `_content/Radzen.Blazor/css/material-base.css` |
| Default 樣式  | `_content/Radzen.Blazor/css/default-base.css`  |
| 深色主題        | `_content/Radzen.Blazor/css/default-dark.css`  |

---

### 10. 元件樣式統一小技巧

* 使用 `<RadzenButton>` 、`<RadzenTextBox>` 、`<RadzenDropDown>` 取代原生元件
* 利用 `Style`，`Class`，`ThemeColor`，`Icon` 等定義視覺統一

---

### 11. 與 Bootstrap 衝突處理

* 樣式錯亂時，考慮移除 Bootstrap 或削減使用
* 元件格式分離：使用 `.rz-` 前置格式群組

---

##
