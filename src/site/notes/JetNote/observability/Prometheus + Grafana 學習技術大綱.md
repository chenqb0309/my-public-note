---
{"dg-publish":true,"permalink":"/jet-note/observability/prometheus-grafana/","title":"Prometheus + Grafana 學習技術大綱","tags":["Prometheus","Grafana","Monitoring","Observability"],"dg-note-properties":{"title":"Prometheus + Grafana 學習技術大綱","tags":["Prometheus","Grafana","Monitoring","Observability"],"created":"2025-09-16"}}
---


# Prometheus + Grafana 學習技術大綱

## 1. Prometheus 基本觀念

### 角色定位

* **Prometheus**：負責「收集與儲存 metrics」
* **Grafana**：負責「可視化與警報」
* 兩者是互補關係，不需要自己寫前端頁面

### Metrics 來源

* 應用程式必須在 `/metrics` endpoint 提供數據
* Prometheus 透過 `scrape_configs` 定期抓取

### Prometheus WebUI ([http://localhost:9090](http://localhost:9090))

* `/metrics`：只顯示 Prometheus 自己的健康狀態
* `/targets`：檢查哪些應用程式被正確抓取
* `Graph`：簡單查詢測試工具，不是完整監控

---

## 2. ASP.NET Core 整合

### 安裝套件

* `prometheus-net.AspNetCore`

### 啟用 Metrics Endpoint

```csharp
app.MapMetrics(); // 預設 http://localhost:5000/metrics
```

### 自定義 Metrics

常用四大類型：Gauge / Counter / Histogram / Summary

#### Label 的概念

* **Label** 是 metrics 的維度，用來區分來源。
* 例如：溫濕度設備的 `device_id` 與 `device_ip`

#### 範例

```csharp
private static readonly Gauge TemperatureGauge =
    Metrics.CreateGauge("device_temperature_celsius", "Temperature", 
        new GaugeConfiguration { LabelNames = new[] { "device_id", "device_ip" } });

TemperatureGauge.WithLabels("sensor1", "10.0.0.10").Set(28.5);
```

### 誤區修正

* ❌ 以為 `9090/metrics` 應該顯示自定義數據
* ✅ 實際上要看 `5000/metrics`（應用程式輸出），Prometheus 只是抓取並存儲

---

## 3. Prometheus 設定

### YAML 設定檔 (`prometheus.yml`)

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: "app-a-metrics"
    static_configs:
      - targets: ["host.docker.internal:5000"]
```

### 誤區修正

* ❌ 嘗試修改容器內的 prometheus.yml
* ✅ 正確做法：建立本地 YAML 檔，掛載進容器 (-v)

### 確認狀態

* [http://localhost:9090/targets](http://localhost:9090/targets) → 確認 target 是 UP
* [http://localhost:9090/graph](http://localhost:9090/graph) → 可以查詢 `device_temperature_celsius`

---

## 4. Grafana 整合

### 角色

* 將 Prometheus 當作資料來源 (Data Source)

### 建立 Panel

* 可用折線圖、儀表盤、直方圖等方式呈現

### 常見映射

* Gauge → 儀表圖、單值顯示
* Counter → 折線圖 (累積值 or `rate()`)
* Histogram → 熱度圖 / 分佈圖
* Summary → 百分位統計圖

---

## 5. 四大 Metrics 類型總結

| 類型            | 特性            | 適用場景         | 常用圖表               |
| ------------- | ------------- | ------------ | ------------------ |
| **Counter**   | 單調遞增，代表事件次數   | API 請求數、錯誤數  | 折線圖 (需搭配 `rate()`) |
| **Gauge**     | 可增可減，代表當前狀態   | CPU 使用率、溫濕度值 | 儀表圖、折線圖            |
| **Histogram** | 計算數值分佈，支援區間統計 | 請求延遲、大小分佈    | 熱度圖、直方圖            |
| **Summary**   | 百分位統計，提供分佈細節  | 延遲 p95/p99   | 折線圖 (百分位曲線)        |

