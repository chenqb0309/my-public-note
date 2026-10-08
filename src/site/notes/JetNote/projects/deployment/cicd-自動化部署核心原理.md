---
{"dg-publish":true,"permalink":"/jet-note/projects/deployment/cicd/","title":"CI/CD 自動化部署核心原理","tags":["CI/CD","GitHub","Gitea","DevOps","部署"],"dg-note-properties":{"title":"CI/CD 自動化部署核心原理","tags":["CI/CD","GitHub","Gitea","DevOps","部署"],"created":"2026-10-06"}}
---


# CI/CD 自動化部署核心原理

> CI/CD 的運作本質上就是**三個角色的互相搭配**，不需要想像得太複雜。
> 本文聚焦「原理與心智模型」，非特定平台的逐步操作。

---

## 一、核心三大組件

| 角色 | 比喻 | 實際載體 | 職責 |
|------|------|----------|------|
| 腳本 (Workflow) | 食譜 / 派工單 | `.github/workflows/deploy.yml` | 規定第一步做什麼、第二步做什麼的純文字指令檔 |
| 介面總部 (GitHub / Gitea) | 餐廳櫃檯 / 派工中心 | 網頁管理介面 | 監聽 `git push` 事件，查表進行任務媒合與派工 |
| 執行器 (Runner) | 無情的工作人 / 廚師 | 雲端虛擬機 或 地端 Linux 程式 | 把程式碼拉下來，照著腳本指令一行行出力執行 |

---

## 二、GitHub 與自架 Gitea 的「工人（Runner）」差異

* **GitHub（公有雲）**：微軟在背後出資，每次 `git push` 時免費在雲端機房當場開機一台全新虛擬電腦幫你執行腳本，跑完立刻銷毀。**因此本機不需安裝 Runner。**
* **Gitea（公司地端）**：公司的小伺服器沒有多餘資源自動開關虛擬機，因此必須在自己的 Linux 上用 Docker 跑一個背景程式（`act_runner`），主動 24 小時向 Gitea 報到掛機等工作。

---

## 三、關鍵細節（打破直覺的業界設計）

### 1. 網路連線方向：是 Runner 主動連向總部

* **誤區**：以為是 GitHub / Gitea 透過 IP / Port 去連線並指揮 Runner。
* **事實**：為了防火牆安全，是 **Runner 從內網主動連出去**聽總部指令。
* **結果**：遠端 Linux **不需要對外開放任何輸入埠（Inbound Port）**，駭客連不進來，安全性極高。

### 2. 派工與專案媒合：看你在「多上層的資料夾」拿 Token（作用域 Scope）

* 啟動地端 Runner 時**完全不需要指定 Repo 名稱**。
* 它透過你在 Gitea 網頁後台複製 Token 的**層級位置**，決定它能看見哪間房間：

| Token 取得層級 | 工人可接的單 |
|----------------|--------------|
| 底層專案 (Repo) | 只能接該專案的單（安全獨立） |
| 中層組織 (Organization) | 可接該部門旗下所有專案的單 |
| 頂層全公司 (Global) | 全公司通用的公用工人 |

### 3. 腳本不寫死在工人身上，而是「隨程式碼一起走」

* 工人（Runner）出廠時是一張白紙，只被寫死一個 SOP：**下載 Repo → 讀取 yml 腳本 → 照辦**。
* 彈性極高：工程師只要在自己電腦改 YAML 檔推上去，部署流程就自動更新，**不需要登入伺服器改設定**。

---

## 四、設計要點摘要

| 面向 | 設計 | 目的 |
|------|------|------|
| 觸發 | 總部監聽 `git push` | 事件驅動的自動部署 |
| 執行 | Runner 拉下 Repo 照腳本跑 | 工人保持無狀態（白紙） |
| 連線方向 | Runner 主動向外連總部 | 免開 Inbound Port，防火牆安全 |
| 權限範圍 | 依 Token 取得層級（Repo / Org / Global） | 一台 Runner 可服務不同範圍的專案 |
| 流程定義 | YAML 隨程式碼版控 | 改流程＝改程式碼並 push，無需登入伺服器 |

---

## 相關連結

* [[JetNote/projects/deployment/Nginx 零耦合 API 網關與藍綠部署設計理解\|Nginx 零耦合 API 網關與藍綠部署設計理解]] — 部署切流的網關層原理
* [[JetNote/projects/deployment/Windows Service 部署流程（實務筆記）\|Windows Service 部署流程（實務筆記）]] — Windows 服務的部署實務
