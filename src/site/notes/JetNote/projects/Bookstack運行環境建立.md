---
{"dg-publish":true,"permalink":"/jet-note/projects/bookstack/","title":"Bookstack運行環境建立","tags":["Bookstack","Docker","Documentation","Self-hosted"],"dg-note-properties":{"title":"Bookstack運行環境建立","tags":["Bookstack","Docker","Documentation","Self-hosted"],"created":"2025-02-06"}}
---


# Bookstack運行環境建立
## Wsl
- 安裝Wsl
    1. Win+R >>> CMD
    2. 安裝wsl
       ```
       wsl --install
    3. 檢查是否安裝成功
       ```
       wsl --list --online
    4. 檢查當前版本
       ```
       wsl -l -v 
       ```
- 啟動
    1. 普通啟動
        a. wsl
        b. 進入安裝時設定的賬號中執行
    2. root啟動
        ``` 
        wsl -u root
        ```
        a. 使用root(管理者權限執行)
        b. 若是需要安裝Docker等需要權限執行的功能,最好使用root執行以正常進行

---

## Docker
- 安裝Docker環境
    1. 下載Docker_DeskTop
    2. 設定Docker環境與Wsl的Ubuntu綁定
     ![image](https://hackmd.io/_uploads/rJ8Zzi-Fke.png)


- Build container
    1. 在docker環境下獲取image
        - 獲取Bookstack
        ```
        docker pull ghcr.io/linuxserver/bookstack
        ```
    2. 運行SQL container
        - 獲取image並運行
        ```
        docker run -d --name bookstack-mysql \
        -e MYSQL_ROOT_PASSWORD=rootpassword \
        -e MYSQL_DATABASE=bookstackdb \
        -e MYSQL_USER=bookstackuser \
        -e MYSQL_PASSWORD=yourpassword \
        -p 3306:3306 \
        mysql:5.7
        ```
    3. 運行Bookstack container
        - 運行Bookstack
        ```
        docker run -d --name bookstack \
        --link bookstack-mysql:mysql \
        -e DB_HOST=bookstack-mysql \
        -e DB_DATABASE=bookstackdb \
        -e DB_USERNAME=bookstackuser \
        -e DB_PASSWORD=yourpassword \
        -p 6875:80 \
        ghcr.io/linuxserver/bookstack
        ```
        a. 執行後可能遇到API_Key錯誤導致的無法正確掛載問題
        b. 生成APIKey後重新設定container及啟動
        - 秘鑰生成
        ```
        docker run -it --rm --entrypoint /bin/bash lscr.io/linuxserver/bookstack:latest appkey
        ```
        - 設定語法(結合在上面的運行Bookstack中一起使用開啟container)
        ```
        -e APP_KEY=youJustBuildedAPIKey \
        ```
        c. 若是進入ExampleDomain頁面,代表掛載成功(但是,這個當前image中的指向域並非我們需要的Bookstack首頁)
        ![image](https://hackmd.io/_uploads/rk5vwjZFJg.png)
        d.此時可以使用以下指令,檢視內部容器網路狀況
        ```
        docker exec -it bookstack bash
        curl http://localhost
        ```
        - 可以檢視到以下結構的信息輸出
        - content處可以看到當前就是指向`example.com`
        ```
        <!DOCTYPE html>
        <html>
            <head>
                <meta charset="UTF-8" />
                <meta http-equiv="refresh" content="0;url='https://example.com/login'" />

                <title>Redirecting to https://example.com/login</title>
            </head>
            <body>
                Redirecting to <a href="https://example.com/login">https://example.com/login</a>.
            </body>
        ```
        e. 此時修改環境變數可以解決此問題
        - 進入 BookStack 容器修改 .env 檔案：
        ```
        docker exec -it bookstack bash
        nano /config/www/.env
        ```
        - 檢查或新增以下內容：
        ```
        APP_URL=http://localhost:6875
        ```
        - 儲存後離開 (Ctrl + O, Enter, Ctrl + X)。
        - 重啟container
        ```
        docker restart bookstack
        ```
    4. 使用映射出來的port就可以使用了
        - 預設賬號: `admin@admin.com`
        - 預設密碼:  `password`
        ```
        http://localhost:6875/login
        ```
        ![image](https://hackmd.io/_uploads/BywVjoZKJe.png)
        
    

        
        