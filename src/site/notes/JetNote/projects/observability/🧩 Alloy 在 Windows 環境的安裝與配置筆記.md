---
{"dg-publish":true,"permalink":"/jet-note/projects/observability/alloy-windows/","title":"🧩 Alloy 在 Windows 環境的安裝與配置筆記","tags":["L-A-G"],"dg-note-properties":{"title":"🧩 Alloy 在 Windows 環境的安裝與配置筆記","tags":["L-A-G"],"created":"2025-10-21"}}
---


# 🧩 Alloy 在 Windows 環境的安裝與配置筆記
（以實際部署經驗與排錯為基礎）

---

## 一、Alloy 安裝概要

### 1️⃣ 安裝方式
- 下載並執行官方 **Alloy for Windows 安裝包（.msi）**
- 安裝完成後系統會自動：
  - 將程式放置於：
    ```text
    C:\Program Files\GrafanaLabs\Alloy
    ```
  - 建立 Windows 服務：
    ```text
    Alloy
    ```
  - 自動啟動該服務並監聽埠口 `:12345`
    ```text
    http://localhost:12345
    ```
    - 若是沒有如預期開啟,則需在regedit界面中設定Arguments
    ``` arguments
    --server.http.listen-addr=LISTEN_ADDR:12345
    ```

---

## 二、服務與系統整合

### 1️⃣ Alloy 服務管理
- 開啟服務管理器：
  ```text
  services.msc
  ```

可用於：

* 啟動 / 停止 / 重新啟動 Alloy
* 確認當前狀態（Running / Stopped）

### 2️⃣ 使用 Regedit 調整 Alloy 設定檔與儲存路徑

**Registry 路徑：**

```
HKEY_LOCAL_MACHINE\SOFTWARE\GrafanaLabs\Alloy
```

**Arguments 範例：**

```text
run
C:\Program Files\GrafanaLabs\Alloy\config.alloy
--storage.path=C:\Program Files\GrafanaLabs\Alloy\Storage
```

**修改步驟：**

1. 開啟 `regedit`
2. 導航至上述路徑
3. 雙擊右側的 **Arguments**
4. 修改 config 路徑
5. 修改Storage路徑 `--storage.path`
    - Storage用於存儲scrape及logTail指針資訊
7. 儲存後重新啟動服務

![image](https://hackmd.io/_uploads/rJSqYsSCee.png)


**注意事項：**

* 確保指定的 `storage.path` 有寫入權限
* 修改前建議匯出 Registry Key 做備份

---

## 三、設定檔管理與路徑規則

### 1️⃣ 設定檔位置

```
C:\Program Files\GrafanaLabs\Alloy\config.alloy
```

若 Registry 指定自訂路徑，則以該檔為準。

### 2️⃣ 路徑規則

* 使用 `/` 而非 `\`
* 建議採用絕對路徑

範例：

```hcl
loki.source.file "logs" {
  targets = [{
    __path__ = "C:/Grafana/logs/app.log"
  }]
}
```

---

## 四、Log 與除錯手段

### 1️⃣ Alloy 執行 Log

* 查看 Windows 事件檢視器：

  ```text
  eventvwr.msc
  Windows Logs → Application → Source = Grafana Alloy
  ```
  ![image](https://hackmd.io/_uploads/BkzXqsBAxl.png)


### 2️⃣ Loki Source 驗證

* 成功運行後，會在指定資料夾建立：

```
loki.source.file.log_scrape
```

### 3️⃣ Alloy Graph 驗證

* 開啟瀏覽器：

```
http://localhost:12345
```

* 檢查模組狀態、Pipeline 拓樸、Source/Target
* 若無資料流動：

  * 路徑錯誤或使用 `\`
  * 權限不足
  * 模組未正確連線

---

## 五、重啟與除錯流程

1️⃣ 停止服務：

```
services.msc → Grafana Alloy → Stop
```

2️⃣ 修改設定檔或 Registry Arguments
3️⃣ 重新啟動服務
4️⃣ 驗證：

* Event Viewer 無錯誤訊息
* Graph 顯示正常
* log_scrape 資料夾出現更新記錄

---

## 六、常見錯誤與排除對應

| 錯誤代碼      | 說明                                        | 解法                  |
| --------- | ----------------------------------------- | ------------------- |
| 1067      | Alloy 啟動失敗，通常是 config 路徑或 storage.path 錯誤 | 檢查 Registry 參數與檔案權限 |
| Graph 無資料 | 路徑錯誤、權限不足或模組未連線                           | 改為絕對路徑 + 使用 `/`     |

---

## 七、實務心得歸納

1. Alloy 安裝後即自動建立服務
2. 使用 services.msc 管理服務，eventvwr.msc 檢查錯誤
3. config.alloy 必須使用 Go 語法路徑格式
4. storage.path 可自訂暫存位置（透過 Registry 調整）
5. 修改設定後必須重新啟動服務
6. 服務預設使用系統帳號執行，需確保有權限讀取指定 log 資料夾

## ※實際配置參考

### 配置文件(config.alloy)
``` config.alloy
// Step 1. 掃描指定目錄下的 log
local.file_match "local_files" {
  //path前綴必須為此__path__""形式,否則Alloy會找不到key
  //路徑遵從設置規範需使用'/'否則會導致路徑匹配失敗
  //可以設定一個附加label=yourSettingName
  path_targets = [{ "__path__" = "C:/Users/jet.chen/source/repos/MES/Source/APP_Agent/Agent.TempMoisDeviceDetector/logs/*.log","labelname"="appName" }]
  sync_period  = "10s"
}

// Step 2. 抓取 log 並傳遞給解析器
loki.source.file "log_scrape" {
  targets       = local.file_match.local_files.targets
  forward_to    = [loki.process.add_label.receiver]
  //設定Alloy開啟時如何抓取log檔 true:從開啟後新增log才抓取 false:file中從頭抓取(可能導致高負荷一般用於本地測試檢視是否成功獲取log)
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
    url = "http://192.168.1.184:3100/loki/api/v1/push"
  }
}

```
## 多Service配置參考
1. 重點在於建立多組爬取管道（scrape-pipline）
2. 其中組件名稱不可互相重複避免抓取錯誤
3. 最後 `write to loki` 可以只有一個(因推送目的地api相同)

``` config.alloy
//=============================================================================//
//======================TempMoisDeviceDetector Log Config======================//
//=============================================================================//

// Step 1. 掃描指定目錄下的 log
local.file_match "local_files" {
  path_targets = [
    { "__path__" = "C:/MES_Service/Agent.TempMoisDeviceDetector/logs/*.log",
      "app"      = "TempMoisDeviceDetector",
    },
  ]
  sync_period = "10s"
}

// Step 2. 抓取 log 並傳遞給解析器
loki.source.file "log_scrape" {
  targets       = local.file_match.local_files.targets
  forward_to    = [loki.process.add_label.receiver]
  tail_from_end = true
}

// Step 3. 解析 Serilog 格式並添加 label
// loki.process 用於提取日誌中的字段並將其作為標籤添加到 Loki 日誌中
loki.process "add_label" {

  // 1️⃣ 將多行 JSON log 合併成一筆
  // Print { ... } 這種情況需要把多行 JSON 當成一個完整事件
  stage.multiline {
    firstline     = "^[0-9]{4}-[0-9]{2}-[0-9]{2}" // 新 log 開頭為時間戳
    max_wait_time = "2s"
  }

  // 2️⃣ 使用正則表達式解析一般 Serilog 格式的 log
  // 若匹配不到（例如 Print { ... } JSON 格式），此 stage 不會丟錯，只是無法建立命名組
  stage.regex {
    expression = "^\\[(?P<timestamp>\\d{4}-\\d{2}-\\d{2}\\s+\\d{2}:\\d{2}:\\d{2}(?:\\.\\d+)?)(?:\\s+[+-]\\d{2}:\\d{2})?\\s+(?P<level>[A-Z]+)\\]\\s+(?P<category>[^:]+):\\s+(?P<message>.*)$"
  }


  // 3️⃣ 嘗試解析 JSON 格式的 log
  // 如果 message 含 JSON 內容，就能提取欄位；否則這步會自動略過
  /*stage.json {
    source = "message"
    expressions = {
      Id           = "Id",
      PrinterName  = "PrinterName",
      LabelType    = "LabelType",
      WorkOrderID  = "WorkOrderID",
      MaterialID   = "MaterialID",
      MaterialName = "MaterialName",
      LotNo        = "LotNo",
      Qty          = "Qty",
    }
  }*/

  // 4️⃣ level 正規化（保持你原有 template 機制）
  stage.template {
    source   = "level"
    template = "{{ .Value | replace \"ERR\" \"error\" | replace \"DBG\" \"debug\" | replace \"INF\" \"info\" | replace \"WRN\" \"warning\" | replace \"FTL\" \"fatal\" }}"
  }

  // 5️⃣ 統一輸出為 Loki 標籤
  stage.labels {
    values = {
      timestamp = "timestamp",
      level     = "level",
      message   = "message",
    }
  }

  forward_to = [loki.write.grafana_loki.receiver]
}

//=============================================================================//
//======================TempMoistureModbusClient Log Config======================//
//=============================================================================//

// Step 1. 掃描指定目錄下的 log
local.file_match "local_files2" {
  path_targets = [
    { "__path__" = "C:/MES_Service/Agent.TempMoistureModbusClient/logs/*.log",
      "app"      = "TempMoistureModbusClient",
    },
  ]
  sync_period = "10s"
}

// Step 2. 抓取 log 並傳遞給解析器
loki.source.file "log_scrape2" {
  targets       = local.file_match.local_files2.targets
  forward_to    = [loki.process.add_label2.receiver]
  tail_from_end = true
}

// Step 3. 解析 Serilog 格式並添加 label
// loki.process 用於提取日誌中的字段並將其作為標籤添加到 Loki 日誌中
loki.process "add_label2" {

  // 1️⃣ 將多行 JSON log 合併成一筆
  // Print { ... } 這種情況需要把多行 JSON 當成一個完整事件
  stage.multiline {
    firstline     = "^[0-9]{4}-[0-9]{2}-[0-9]{2}" // 新 log 開頭為時間戳
    max_wait_time = "2s"
  }

  // 2️⃣ 使用正則表達式解析一般 Serilog 格式的 log
  // 若匹配不到（例如 Print { ... } JSON 格式），此 stage 不會丟錯，只是無法建立命名組
  stage.regex {
    expression = "^\\[(?P<timestamp>\\d{4}-\\d{2}-\\d{2}\\s+\\d{2}:\\d{2}:\\d{2}(?:\\.\\d+)?)(?:\\s+[+-]\\d{2}:\\d{2})?\\s+(?P<level>[A-Z]+)\\]\\s+(?P<category>[^:]+):\\s+(?P<message>.*)$"
  }


  // 3️⃣ 嘗試解析 JSON 格式的 log
  // 如果 message 含 JSON 內容，就能提取欄位；否則這步會自動略過
  /*stage.json {
    source = "message"
    expressions = {
      Id           = "Id",
      PrinterName  = "PrinterName",
      LabelType    = "LabelType",
      WorkOrderID  = "WorkOrderID",
      MaterialID   = "MaterialID",
      MaterialName = "MaterialName",
      LotNo        = "LotNo",
      Qty          = "Qty",
    }
  }*/

  // 4️⃣ level 正規化（保持你原有 template 機制）
  stage.template {
    source   = "level"
    template = "{{ .Value | replace \"ERR\" \"error\" | replace \"DBG\" \"debug\" | replace \"INF\" \"info\" | replace \"WRN\" \"warning\" | replace \"FTL\" \"fatal\" }}"
  }

  // 5️⃣ 統一輸出為 Loki 標籤
  stage.labels {
    values = {
      timestamp = "timestamp",
      level     = "level",
      message   = "message",
    }
  }

  forward_to = [loki.write.grafana_loki.receiver]
}


//======================Common Loki Write Config======================//
// Step 4. 寫入 Loki
loki.write "grafana_loki" {
  endpoint {
    url = "http://192.168.1.184:3100/loki/api/v1/push"
  }
}

```