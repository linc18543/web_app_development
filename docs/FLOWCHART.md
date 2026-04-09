# 系統流程圖文件 (Flowchart) - 個人記帳簿

本文件依據 PRD 與架構設計，繪製出了使用者的操作流程圖以及新增一筆記帳資料的系統序列圖。

## 1. 使用者流程圖 (User Flow)

這張流程圖展示了使用者進入個人記帳簿系統後的主要操作路徑與分支畫面。

```mermaid
flowchart LR
    Start([使用者開啟網頁]) --> CheckLogin{是否已登入？}
    
    CheckLogin -->|否| Auth[登入 / 註冊頁面]
    Auth --> CheckLogin
    
    CheckLogin -->|是| Dashboard[儀表板 / 首頁]
    
    Dashboard --> Nav{要執行什麼操作？}
    
    Nav -->|瀏覽統計| Stats[檢視圖表分析與預算進度]
    Nav -->|新增收支| Add[填寫收支表單]
    Nav -->|管理紀錄| List[收支歷史紀錄列表]
    Nav -->|設定預算| Budget[預算設定頁面]
    
    Add -->|送出表單| SaveDb[(儲存紀錄)]
    SaveDb -->|返回| List
    
    List --> EditDelete{編輯或刪除？}
    EditDelete -->|編輯| Edit[修改表單] --> SaveDb
    EditDelete -->|刪除| DeleteDb[(刪除資料)] --> List

    Budget -->|儲存| SaveBudget[(儲存預算)] --> Dashboard
```

## 2. 系統序列圖 (Sequence Diagram)

這張序列圖具體描述了「使用者在畫面上新增一筆記帳紀錄」背後的系統互動流程，涵蓋從前端瀏覽器發出請求到資料庫寫入的完整細節。

```mermaid
sequenceDiagram
    actor User as 使用者
    participant Browser as 瀏覽器
    participant Flask as Flask (Route)
    participant Model as Transaction (Model)
    participant DB as SQLite (Database)

    User->>Browser: 在「新增收支」頁面填寫金額與分類後點擊送出
    Browser->>Flask: 發送 POST /record/add (表單資料)
    
    Flask->>Flask: 驗證使用者是否已登入？
    Flask->>Flask: 驗證表單資料格式 (如金額大於0)
    
    alt 資料格式有誤
        Flask-->>Browser: 回傳錯誤訊息與原表單頁面
        Browser-->>User: 顯示錯誤提示
    else 資料正確
        Flask->>Model: 要求建立新紀錄 (Create)
        Model->>DB: 執行 SQL INSERT
        DB-->>Model: 回傳寫入成功
        Model-->>Flask: 紀錄新增完成
        Flask-->>Browser: HTTP 302 重導向到收支列表頁
        Browser-->>User: 顯示「新增成功」並看見最新紀錄
    end
```

## 3. 功能清單對照表

以下為本專案 MVP 及預期擴充功能的網址路由 (URL) 與 HTTP 方法對照表，作為後續 Flask 實作與 API 設計的參考依據。

| 功能名稱 | URL 路徑 | HTTP 方法 | 說明 |
| :--- | :--- | :--- | :--- |
| **登入頁面與送出** | `/auth/login` | GET / POST | 顯示登入表單 / 驗證登入資訊 |
| **註冊頁面與送出** | `/auth/register` | GET / POST | 顯示註冊表單 / 建立新帳號 |
| **登出** | `/auth/logout` | GET | 清除 Session 並導向登入頁 |
| **首頁儀表板** | `/dashboard` | GET | 顯示總收支統計、圓餅圖與預算進度 |
| **收支清單** | `/record/` | GET | 列出歷史收支紀錄 (可支援分頁/篩選) |
| **新增收支** | `/record/add` | GET / POST | 顯示新增表單 / 儲存新紀錄到資料庫 |
| **編輯收支** | `/record/edit/<id>` | GET / POST | 顯示某筆紀錄的編輯表單 / 更新該筆資料 |
| **刪除收支** | `/record/delete/<id>`| POST | 將指定 `<id>` 的收支紀錄從資料庫移除 |
| **預算設定** | `/budget` | GET / POST | 顯示預算設定表單 / 儲存月份預算設定 |
| **匯出報表** | `/export` | GET | (Nice to Have) 下載所有的收支紀錄為 CSV 檔 |
