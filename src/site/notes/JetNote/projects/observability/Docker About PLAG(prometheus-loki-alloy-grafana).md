---
{"dg-publish":true,"permalink":"/jet-note/projects/observability/docker-about-plag-prometheus-loki-alloy-grafana/","title":"Docker About PLAG(prometheus-loki-alloy-grafana)","tags":["L-A-G","Docker"],"dg-note-properties":{"title":"Docker About PLAG(prometheus-loki-alloy-grafana)","tags":["L-A-G","Docker"],"created":"2025-10-20"}}
---


# Docker About PLAG(prometheus-loki-alloy-grafana)

### Prometheus.yaml
``` prometheus
global:
  scrape_interval: 15s        # 預設每 15 秒抓取一次
  evaluation_interval: 15s    # 預設每 15 秒評估一次規則 (alert / recording rules)

scrape_configs:

  # 若是沒有使用scrape_interval，則會使用 global 的設定
  - job_name: "app-a-metrics"
    scrape_interval: 20s       # 覆蓋 global 設定，改成每 20 秒抓一次
    static_configs:
      - targets: ["host.docker.internal:5000"]  # 目標 IP 和端口
```

### Loki(compactor)
``` loki
compactor:
  working_directory: /loki/compactor
  shared_store: filesystem
  retention_enabled: true  
  # ↑ 記得在compactor加上這一行,否則會導致記錄始終只有半個小時可以被搜索到
```

### Alloy-config.hcl

``` alloy
// Step 1. 掃描指定目錄下的 log
local.file_match "local_files" {
  path_targets = [{ "__path__" = "/var/log/agent/*.log" }]
  sync_period  = "10s"
}

// Step 2. 抓取 log 並傳遞給解析器
loki.source.file "log_scrape" {
  targets       = local.file_match.local_files.targets
  forward_to    = [loki.process.add_label.receiver]
  tail_from_end = false
}

// Step 3. 解析 Serilog 格式並添加 label
// loki.process 用於提取日誌中的字段並將其作為標籤添加到 Loki 日誌中
loki.process "add_label" {

  // 使用正則表達式解析 Serilog 日誌格式,此時生成一個變數字典(還未成為標籤)
  // ?P<> 用於命名捕獲組
  stage.regex {
    expression = "^\\[(?P<timestamp>[0-9\\-]+\\s+[0-9:\\.]+)\\s+(?P<timezone>[+\\-0-9:]+)\\s+(?P<level>[A-Z]+)\\]\\s+(?P<category>[^:]+):\\s+(?P<message>.*)$"
  }

  //使用stage.regex,因此不能直接使用stage.replace來修改日誌內容
  //最終解法使用stage.template來重新構建日誌內容
  //template語法參考Go模板語法,重要事項: `{{}}`語法不能使用容易有錯誤,直接使用pipe管道符號來進行多重替換
  stage.template {
    source   = "level"
    template = "{{ .Value | replace \"ERR\" \"error\" | replace \"DBG\" \"debug\" | replace \"INF\" \"info\" | replace \"WRN\" \"warning\" | replace \"FTL\" \"fatal\" }}"
  }

  // 將變數字典中的字段轉換為 Loki 標籤
  stage.labels {
    values = {
      timestamp = "timestamp",
      level     = "level",
      category  = "category",
      message   = "message",
    }
  }

  forward_to = [loki.write.grafana_loki.receiver]
}

// Step 4. 寫入 Loki
loki.write "grafana_loki" {
  endpoint {
    url = "http://loki:3100/loki/api/v1/push"
  }
}


```

---

### docker-compose

``` docker-compose
services:
  prometheus:
    image: prom/prometheus:v3.5.0
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus-data:/prometheus
    ports:
      - "9090:9090"
    restart: unless-stopped

  loki:
    image: grafana/loki:2.9.0
    container_name: loki
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml #因為loki沒有預設的entrypoint，所以要用command指定啟動參數
    restart: unless-stopped
    #掛載設定檔用於同步相關設定
    #掛載localtime用於同步宿主機時區,否則容易發生於Grafana時間段落錯位問題(因Grafana會自動同步)
    volumes:
      - ./loki-config.yaml:/etc/loki/local-config.yaml
      - /etc/localtime:/etc/localtime:ro


  alloy:
    image: grafana/alloy:latest
    container_name: alloy
    ports:
      - "12345:12345" #alloy的web介面
    #因為alloy沒有預設的entrypoint，所以要用command指定啟動參數
    #同時因為需要使用alloy的web介面來查看目前的狀態，所以要把server.http.listen-addr設定成
    command: run /etc/alloy/config.alloy --server.http.listen-addr=0.0.0.0:12345 
    restart: unless-stopped
    volumes:
      - ./alloy-config.hcl:/etc/alloy/config.alloy #alloy的設定檔
      #比較特別的部分是,因為alloy要讀取log檔案,所以要把log檔案的路徑mount進去
      - C:/App/LogSource-A/logs:/var/log/agent


  grafana:
    image: grafana/grafana:12.1.1
    container_name: grafana
    volumes:
      - ./grafana-data:/var/lib/grafana
    ports:
      - "3000:3000"
    restart: unless-stopped

    

```