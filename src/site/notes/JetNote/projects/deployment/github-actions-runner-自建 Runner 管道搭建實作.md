---
{"dg-publish":true,"permalink":"/jet-note/projects/deployment/github-actions-runner-runner/","title":"自建 GitHub Actions Runner 管道搭建（實作篇）","tags":["CI/CD","GitHub Actions","Runner","部署","維運"],"dg-note-properties":{"title":"自建 GitHub Actions Runner 管道搭建（實作篇）","tags":["CI/CD","GitHub Actions","Runner","部署","維運"],"created":"2026-10-06"}}
---


# 自建 GitHub Actions Runner 管道搭建（實作篇）

> 對象：**GitHub 專案 ↔ 遠端 Linux 實體機** 之間的自動化管道（Pipeline）架設。
> 原理與心智模型請見 [[JetNote/projects/deployment/cicd-自動化部署核心原理\|cicd-自動化部署核心原理]]；本篇為逐步實作紀錄。

---

## 步驟 0：前置作業 —— 獲取官方客製化指令表

不要死背下載網址與認證金鑰。GitHub 會依你的平台與架構，自動生成一份含**最新穩定版本號**與**專屬 Token 語法**的指令懶人包：

1. 進入 GitHub 專案倉庫（Repository）。
2. 導覽列點 **Settings** ➡️ 左側 **Actions** ➡️ 展開 **Runners**。
3. 點右上角藍色按鈕 **New self-hosted runner**。
4. 平台選 **Linux**、架構選 **X64**（依伺服器硬體而定），下方即生成步驟 2、3 的精準指令。

> 心法：**以下步驟 2、3 的網址與 Token，一律從這個面板複製**，不要憑記憶手打。

---

## 步驟 1：在遠端 Linux 建立專屬目錄

```bash
# 在 jet 帳號家目錄下建立 actions-runner 資料夾並進入
mkdir ~/actions-runner && cd ~/actions-runner
```

---

## 步驟 2：下載與解壓 Runner 工具包（對照步驟 0 面板）

```bash
# 1. 下載官方壓縮包（版本號依步驟 0 面板顯示為準，勿手打）
curl -o actions-runner-linux-x64.tar.gz -L \
  https://github.com/actions/runner/releases/download/v<VERSION>/actions-runner-linux-x64-<VERSION>.tar.gz

# 2. 解壓縮，釋出內建的安裝與運行腳本（config.sh、run.sh、svc.sh）
tar -xzf ./actions-runner-linux-x64.tar.gz
```

---

## 步驟 3：金鑰認證與綁定（對照步驟 0 面板）

執行 `config.sh`，讓這台 Linux 實體機與 GitHub 倉庫建立加密連線憑證：

```bash
# Token 由步驟 0 面板自動帶入；過程中可自訂 Runner 名稱（例：jet）
./config.sh --url https://github.com/<owner>/<repo> --token <面板產生的 Token>
```

設定過程會依序詢問（直接 Enter 用預設即可）：

| 提問 | 說明 |
|------|------|
| `Enter name of the runner` | Runner 名稱（例：`jet`） |
| `Enter name of runner group` | 群組（預設 `Default`） |
| `Enter name of labels` | 標籤（預設 `self-hosted,Linux,X64`） |
| `Enter name of work folder` | 工作目錄（預設 `_work`） |

> ⚠️ **Token 是一次性且有時效**（約 1 小時）。過期或重複使用會失敗，需回步驟 0 面板重新產生。

---

## 步驟 4：註冊為 Linux 系統服務

為避免 Runner 佔用終端機畫面，並確保它 24 小時待命、開機自啟，用內建 `svc.sh` 嵌入系統背景（需 systemd 與 sudo 權限）：

```bash
# 1. 登記註冊：以管理員權限將 Runner 寫入系統服務清單
sudo ./svc.sh install

# 2. 正式開工：背景啟動服務。此時 GitHub 面板的 Runner 狀態會亮起 🟢 Idle
sudo ./svc.sh start
```

---

## 步驟 5：撰寫 Workflow（專案根目錄 `.github/workflows/deploy.yml`）

在本機專案中建立 `.github/workflows/deploy.yml`，定義自動化任務並指定由自建 Runner 接單：

```yaml
name: Deploy

# 觸發時機：有人 push 程式碼到 main 分支
on:
  push:
    branches:
      - main

jobs:
  deploy-and-switch:
    # 核心綁定：指定帶有 self-hosted 標籤的自建 Runner
    runs-on: self-hosted

    steps:
      # 步驟一：官方 checkout，讓 Runner 主動把最新程式碼拉回 Linux 工作目錄
      - name: 1. 檢查並拉取最新程式碼
        uses: actions/checkout@v4

      # 步驟二：執行本地 CLI 測試
      - name: 2. Say "Hello, World!"
        run: |
          echo "Hello, World! This is a test of the CI/CD pipeline."
```

> 更精準的綁定可寫 `runs-on: [self-hosted, linux, x64]`，避免派到標籤不合的 Runner。

---

## 步驟 6：Git Push 觸發並驗收

```bash
git add .
git commit -m "Pipeline 管道搭建完成測試"
git push origin main
```

Runner 透過 Polling 領取任務，並把 CLI 輸出即時串流回 GitHub 網頁。

**驗收**：GitHub 專案網頁 ➡️ **Actions** ➡️ 點進 `deploy-and-switch` 工作 ➡️ 展開步驟 2，即可看到 Linux 實體機背景印出的 CLI 日誌。

---

## 注意事項與常見坑

| 項目 | 說明 |
|------|------|
| **Token 時效** | 面板產生的 `--token` 一次性、約 1 小時內有效，過期需重新產生 |
| **安全性** | 自架 Runner **不建議掛在 public repo**：任何人送 PR 都可能在你機器上執行程式碼 |
| **連線方向** | Runner 主動向外連 GitHub，**不需開任何 Inbound Port**（見原理篇） |
| **系統依賴** | `svc.sh` 依賴 systemd；部分發行版首次執行前需 `./bin/installdependencies.sh` 補齊相依 |
| **權限範圍** | Runner 能看見哪些專案，取決於 config Token 取得的層級（Repo / Org / Global），與原理篇一致 |
| **標籤綁定** | `runs-on` 建議用 `[self-hosted, linux, x64]` 明確指定，避免派錯機器 |

---

## 相關連結

* [[cicd-自動化部署核心原理]] — 三大組件、Runner 連線方向、Token 作用域的心智模型
* [[docker-container-host-connectivity-容器與實體主機的三大聯通]] — 若 Runner 以容器形式運行時的聯通觀念
