---
{"dg-publish":true,"permalink":"/jet-note/dotnet/dotnet-build-vs-publish/","title":".NET Build vs Publish 筆記（工程觀點）","tags":[".Net"],"dg-note-properties":{"title":".NET Build vs Publish 筆記（工程觀點）","tags":[".Net"],"created":"2025-07-17"}}
---


# .NET Build vs Publish 筆記（工程觀點）

## 🎯 主題定義

在 .NET 專案開發與部署中，常見初學者誤用「Build」作為正式部署依據，其實這樣做並不可靠。本筆記釐清 `Build` 與 `Publish` 的正確使用情境，並補充同層級的關鍵概念。

---

## 🧱 Build 與 Publish 的根本差異

| 面向                 | Build                       | Publish                    |
| ------------------ | --------------------------- | -------------------------- |
| 目標                 | 產生可以在本機執行的中繼產物              | 產生可直接部署到正式環境的最終成品          |
| 預設輸出路徑             | `bin/Debug` 或 `bin/Release` | `bin/Release/netX/publish` |
| 是否包含靜態檔案與 Razor 編譯 | 不一定完整                       | 會處理並包含完整資源與預編譯產物           |
| 是否移除開發用檔案          | 否（會有 .pdb, 測試用檔）            | 是（保留正式運行所需檔案）              |
| 適合用途               | 本機開發、測試、除錯                  | 正式部署、交付、CI/CD              |

---

## ❌ 錯誤觀念：Release Build = 正式產品

雖然 `Build -c Release` 產出的結果執行效能較佳，但它並不等於「正式產品」。

* 它可能仍含有開發環境用的資源
* 設定檔可能載入錯誤（見下節）
* 不保證所有必要資源與相依檔案齊全

---

## ✅ ASPNETCORE\_ENVIRONMENT 與設定行為差異

.NET Web 與 Background 專案會根據 `ASPNETCORE_ENVIRONMENT` 載入對應的設定檔，例如：

* `appsettings.Development.json`
* `appsettings.Production.json`

若你用 build 而未切換正確環境變數：

* 你可能部署了開發版資料庫連線字串
* 日誌輸出可能太多（如 LogLevel=Debug）
* 假登入 / stub 資料仍在生效中

🔧 正確方式是搭配 `publish` 時切換至 `Production` 環境進行完整測試與產出。

---

## 🧠 工程語意：Build 是構建，Publish 是交付

| 項目   | Build       | Publish           |
| ---- | ----------- | ----------------- |
| 工程語意 | 我準備好執行或測試它  | 我要交出去讓別人用它        |
| 屬性   | 組裝階段        | 發佈階段              |
| 品質保證 | 未經完整驗證，可能缺漏 | 通常會經過環境切換、最小打包與驗證 |

> ✅ 若你是負責部署的工程師，應該一律以 Publish 為正式交付基準。

---

## 🏁 建議工作流程

```bash
# 建議部署產出流程：
dotnet publish -c Release -o ./deploy

# 搭配 ASPNETCORE_ENVIRONMENT
SET ASPNETCORE_ENVIRONMENT=Production
```

* 永遠不要直接部署 `bin/Release` 資料夾
* Publish 出來的內容才是穩定、完整、可部署的成品

---

## ✅ 總結原則

1. `Release Build` 並不是部署產物，`Publish` 才是
2. `ASPNETCORE_ENVIRONMENT` 決定設定檔與執行行為
3. `Build` 是給開發者用來測試的中繼品
4. `Publish` 是給正式環境的交付品
5. 工程團隊應統一部署流程：始終使用 `dotnet publish`

> 📌 建立一個清晰、可預期、可複製的部署流程，是穩定產品品質的基石。
