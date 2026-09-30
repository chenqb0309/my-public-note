---
{"dg-publish":true,"permalink":"/jet-note/dotnet/entity-framework/","title":"Entity Framework","tags":["Entity-Framework","ORM",".NET"],"dg-note-properties":{"title":"Entity Framework","tags":["Entity-Framework","ORM",".NET"],"created":"2025-02-19"}}
---


# Entity Framework

## 什麼是EF

- Microsoft體系下,使用面向物件的方式操作SQL的一種方法
- 透過 DbContext + LINQ + SQL Provider將需求的操作轉換成SQL指令執行

## 使用方法概述

- 創建Model,並使用```data annotation```的方式標註對table的設定(此為基本用法)
- 使用定義的語法,在隔離底層代碼的情況下,連線資料庫並操作相關的CRUD動作

## 文章參考

[Entity Framework](https://www.cnblogs.com/cqpanda/p/16768001.html)
