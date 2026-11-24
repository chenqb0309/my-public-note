---
{"dg-publish":true,"permalink":"/jet-note/docker/docker-container-host-connectivity/","title":"Docker 容器與實體主機的三大聯通（Volume / Port / Gateway）","tags":["Docker","Container","網路","反向代理","Nginx"],"dg-note-properties":{"title":"Docker 容器與實體主機的三大聯通（Volume / Port / Gateway）","tags":["Docker","Container","網路","反向代理","Nginx"],"created":"2026-10-06"}}
---


# Docker 容器與實體主機的三大聯通（Volume / Port / Gateway）

> **核心**：容器與實體主機之間有三種「聯通綁定」，各自對應一個**方向性**。
> 掌握這三條線的流向，就掌握了容器與外部環境互動的全貌。
>
> 本文以「Nginx 跑在容器、後端服務跑在實體主機」的單機情境作為範例，
> 但真正的主體是這三種聯通本身。藍綠切換僅為應用場景之一，深入原理請見 [[JetNote/projects/deployment/Nginx 零耦合 API 網關與藍綠部署設計理解\|Nginx 零耦合 API 網關與藍綠部署設計理解]]。

---

## 一、三種聯通一覽

```
[ 外部使用者 ]                                       [ 實體 Linux 主機 ]
      │                                                     │
      │ (1) Port 聯通 (由外向內)                             │
      ▼                                                     ▼
┌────────────────────────────────────────┐          ┌───────────────────┐
│ Docker 容器 (my-nginx-proxy)            │          │  舊版藍色服務 (8081) │
│                                        │          │                   │
│   ┌──────────────┐                     │          │  新版綠色服務 (8082) │
│   │  nginx.conf  │◄────────────────────┼──────────┼───────────────────┘
│   └──────────────┘ (2) Volume 聯通      │          │
│          │             (外部環境聯通給內部)│          │
│          │                             │          │
│          ▼ (3) Gateway 聯通 (由內向外)   │          │
│   [ host.docker.internal ] ────────────┼──────────►
│        (虛擬出海口網關)                 │
└────────────────────────────────────────┘
```

| 綁定 | 本質 | 流向 |
|------|------|------|
| Volume | 空間的同步 | 外部 ➡️ 內部 |
| Port | 連接埠的對應 | 外部 ➡️ 內部 |
| Add Host / Gateway | 網路的突圍 | 內部 ➡️ 外部 |

---

## 二、逐一拆解

### 1. Volume 聯通：空間的同步（外部 ➡️ 內部）

* **本質**：將實體機的檔案「聯通」給 Docker 內部使用。
* **流向**：當容器內的程式啟動並讀取檔案時，它看見的是外部環境提供的實體檔案。
* **效益**：在外部用 VS Code 改設定，容器內部立即同步，不需重建映像檔或重新複製檔案。

### 2. Port 聯通：連接埠的對應（外部 ➡️ 內部）

* **本質**：由外向內的網路連線。
* **流向**：外部使用者呼叫 `Server IP:Port`，流量順著這條「一對一延長線」導入 Docker 內部，由容器內服務接收。

### 3. Add Host / Gateway 聯通：網路的突圍（內部 ➡️ 外部）

* **本質**：由內向外的反向呼叫，解決單機（同 IP）環境的「孤島效應」。
* **流向**：容器內服務接收流量後，透過「虛擬出海口（Gateway）」把流量反向倒出去，連回實體機上的特定 Port。
* **思考例外**：若後端服務位於**不同 IP** 的獨立電腦上，容器可直接走真實網路連線，不需此 Gateway 密道設定。

---

## 三、實作範例：單機 Nginx 反向代理

### 1. 外部聯通設定檔 `~/nginx-proxy/nginx.conf`

在 Linux 實體主機上建立設定檔，用 `host.docker.internal` 域名指向實體機的服務連接埠。

```nginx
events { worker_connections 1024; }

http {
    upstream backend_servers {
        # 預設導向實體機跑在 8081 的服務
        server host.docker.internal:8081;

        # 切換時，改為下行（實體機跑在 8082 的另一版本）
        # server host.docker.internal:8082;
    }

    server {
        listen 80;
        server_name localhost;

        location / {
            proxy_pass http://backend_servers;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

### 2. 啟動容器（由 VS Code 遠端終端機執行）

```bash
docker run -d \
  --name my-nginx-proxy \
  -p 80:80 \
  -v /home/jet/nginx-proxy/nginx.conf:/etc/nginx/nginx.conf:ro \
  --add-host=host.docker.internal:host-gateway \
  --restart always \
  nginx:latest
```

### 3. 參數魔鬼細節

三種聯通各自對應的參數：

| 參數 | 對應聯通 | 說明 |
|------|----------|------|
| `-v ...:ro` | Volume | 檔案掛載。外部設定檔傳給內部，`:ro`（Read-Only）確保容器無法篡改實體機檔案（格式：`外部:內部`） |
| `-p 80:80` | Port | 連接埠對應。實體機 Port 80 → 容器 Port 80（格式：`外部:內部`） |
| `--add-host=host.docker.internal:host-gateway` | Gateway | 格式反轉例外（`網域名稱:實體機IP`）。在容器內部電話簿建立紀錄，將自訂域名指向外部實體主機的閘道器 |
| `-d` | — | Detached 背景模式。容器在背景運行，關閉終端機也不會中斷 |
| `--restart always` | — | 實體伺服器意外重啟時，容器自動跟著醒來，無需手動重開 |

---

## 四、設計要點摘要

| 面向 | 作法 | 目的 |
|------|------|------|
| 設定同步 | Volume 掛載 `nginx.conf` | 外部改檔即時生效，容器不需重建 |
| 對外入口 | Port `80:80` 對應 | 外部流量導入容器內服務 |
| 回呼實體機 | `host.docker.internal` + `--add-host` | 解決同 IP 孤島，讓容器內服務能連回實體機 Port |
| 開機自啟 | `--restart always` | 主機重啟後容器自動恢復 |

---

## 相關連結

* [[JetNote/docker/About Docker\|About Docker]] — Docker / Container / Compose 基礎觀念
* [[JetNote/projects/deployment/Nginx 零耦合 API 網關與藍綠部署設計理解\|Nginx 零耦合 API 網關與藍綠部署設計理解]] — 網關路由與藍綠切換的原理層（本文範例的延伸）
