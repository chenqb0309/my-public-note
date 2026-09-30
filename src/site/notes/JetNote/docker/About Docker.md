---
{"dg-publish":true,"permalink":"/jet-note/docker/about-docker/","title":"About Docker","tags":["Docker","Container"],"dg-note-properties":{"title":"About Docker","tags":["Docker","Container"],"created":"2025-02-14"}}
---


# About Docker

## 什麼是Docker

1. Docker
    a. Docker 是一個容器管理平台，允許開發者將應用與它的所有依賴封裝成輕量級的容器。
    b. Container 是Docker中的基礎單位,通常運行一個獨立的應用或服務,同時,這些容器可以互相協作,形成完整的系統架構。
    c. Images 映像檔是Build Container的基礎,相當於該程式或是服務當下狀態的<快照>,在運行Container時,將這個快照一比一還原即可。

2. Docker的運行
    ![Docker指令圖解](https://hackmd.io/_uploads/SyeAdGecye.png)

3. Dockerfile
    a. 當需要建立映像檔（Image）時，必須透過 Dockerfile 告知 Docker 應如何建構環境，主要包含以下內容：

      1. 基礎環境（選擇適合的 Base Image，如 ubuntu, node, alpine）
      2. 專案文件（使用 COPY 或 ADD 指定要加入映像檔的檔案）
      3. 執行指令（如安裝依賴、設定環境變數、啟動應用程式）

    b. 以上步驟皆透過 Dockerfile 定義，當執行生成指令時。

```docker!
docker build -t <image_name> . 
```

   **Docker 會依照文件中的指令逐步執行，最終生成符合設計的映像檔（Image）。**
  
---
  
## 什麼是Docker-Compose

- Docker Compose 是一種工具，允許使用 YAML 文件來定義多個容器的組合與管理。
- 透過一個 docker-compose.yml 文件，一次性定義所有要運行的容器及其關聯。
- 進一步補充：
  - Docker-Compose避免為了開啟多個Container而手動執行多次docker run的問題:

```docker_Compose!
docker-compose.yml
```

  一鍵開啟所有你設定的Container。

常見的 docker-compose.yml 設定項目

  1. services：定義不同的容器（例如 Web Server、Database）
  2. volumes：定義資料存放位置，避免容器刪除時資料遺失
  3. networks：設定不同容器之間的通訊方式
  4. environment：設定環境變數，例如 DB 帳號密碼

---

總結: Docker Compose 主要用於本地開發環境
適合本地測試或小型專案的部署
在生產環境通常會使用 Kubernetes（K8s） 來取代
