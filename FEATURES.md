# 🔧 功能需求與技術規格書 (Feature Log & Technical Specifications)

> **核心定位**：記錄已實現的功能列表，並定義網頁開發的 HTML/CSS DOM 結構、JavaScript 邏輯與 API 整合規格。  
> 旅遊規劃與 SOP 指南請參閱 [`BLUEPRINT.md`](./BLUEPRINT.md)。

---

## 1. ✅ 已實現功能清單

### 🎨 UI / 視覺設計
- [x] 參考美西版本 (2026-la.vercel.app) 的整體視覺風格
- [x] 日本版：Sakura 粉色主題 + 飄落櫻花動畫 (`🌸`)
- [x] 九州版：Autumn 橘色主題 + 飄落楓葉動畫 (`🍁🍂`)
- [x] 白色卡片 + 毛玻璃效果 (`backdrop-blur`)
- [x] 漸層背景 (linear-gradient)
- [x] 圓角 Tab 導航列（行程 / 訂單資訊 / 自駕 / 購物 / 行李 / 注意事項）
- [x] 日期選擇器（橫向捲動日期 Pill，含左右箭頭）
- [x] 卡片 hover 微動畫效果
- [x] 響應式設計 (RWD)，支援手機 / 平板 / 桌機
- [x] PWA 支援（`apple-mobile-web-app-capable`、`theme-color`）

### 📅 行程功能
- [x] 每日行程卡片，點擊日期切換當天內容
- [x] 事件卡片支援多種類型：`food` / `sight` / `buy` / `booking` / `transport` / `flight` / `hotel`
- [x] 彩色 Badges 標籤（食 / 景 / 購 / 訂 / 交通等）
- [x] `tip` 可展開/收合的藍色詳細攻略框 (`fas fa-info-circle`)
- [x] `notice` 雙色警告框（特別重要為紅字邊框 / 一般警告為黃字邊框，底色皆為淡米色）
- [x] 上一天 / 下一天按鈕導航
- [x] 自動跳轉到「今天」的日期（若在旅程期間）

### 🌦️ 天氣功能
- [x] 整合 Open-Meteo API 取得即時 16 天天氣預報
- [x] 每個城市/每天配有對應座標 (`dayCoordinates`)
- [x] 天氣顯示可點擊，連結到 tenki.jp 詳細預報頁
- [x] 超出預報範圍時顯示「歷史均溫」
- [x] 天氣快取機制（避免重複 API 請求）

### 🛒 購物清單
- [x] 分類清單（藥妝 / 零食 / 動漫 / 服飾等）
- [x] 每項可打勾確認，無預填項目（使用者自訂）

### 🧳 行李清單
- [x] 分類清單（證件 / 金錢 / 衣物 / 藥品 / 3C / 盥洗）
- [x] 每項可打勾確認，根據季節/成員/目的地動態推薦

### ✅ 注意事項代辦清單
- [x] 分階段提醒（緊急 / 出發前 1-2 個月 / 出發前 1-2 週 / 機上）
- [x] **勾選後自動儲存進度** (`localStorage`)，重開網頁不會消失
- [x] 「緊急」紅色 Badge 標記，完全與行程事件預訂關聯生成

---

## 2. 🩻 HTML/CSS 頁面架構規格

網頁採用單一檔案單頁式應用（Single Page Application, SPA），以下為主要 DOM 結構：

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>旅程名稱</title>
    <!-- Tailwind CSS & FontAwesome -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <style>
        /* 定義字體、橫向捲動條隱藏與葉片下落動畫 */
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
        @keyframes fall {
            0%   { transform: translateY(-20px) rotate(0deg); opacity: 1; }
            100% { transform: translateY(100vh) rotate(420deg); opacity: 0; }
        }
        .leaf {
            position: fixed; top: -20px; font-size: 1.2rem;
            animation: fall linear infinite; pointer-events: none; z-index: 0;
        }
    </style>
</head>
<body class="text-gray-800 pb-16 relative">
    <!-- Header 區塊 (包含標題、副標題與 Tab 切換列) -->
    <header class="pt-6 pb-3 text-center px-3 relative z-10">...</header>

    <!-- 行程檢視區塊 (View 1) -->
    <main id="view-itinerary" class="block relative z-10">
        <!-- 橫向日期導航列 -->
        <div id="date-nav">...</div>
        <!-- 每日行程主卡片 -->
        <div id="main-card-content">
            <!-- 動態生成事件清單 -->
            <div id="card-events" class="space-y-4"></div>
        </div>
    </main>

    <!-- 其他檢視區塊 (預設隱藏 class="hidden") -->
    <section id="view-tools" class="hidden">...</section>
    <section id="view-transport" class="hidden">...</section>
    <section id="view-shopping" class="hidden">...</section>
    <section id="view-luggage" class="hidden">...</section>
    <section id="view-notes" class="hidden">...</section>
</body>
</html>
```

---

## 3. ⚙️ JavaScript 核心控制邏輯

### 3.1 視圖切換 (Tab Navigation)
透過控制 `hidden` 與 `block` 類別，實現秒級無縫切換視圖：
```javascript
function switchTab(tabId) {
    const views = ['itinerary','tools','transport','shopping','luggage','notes'];
    views.forEach(v => {
        document.getElementById('view-' + v).classList.replace('block', 'hidden');
        document.getElementById('tab-' + v).classList.remove('active');
    });
    document.getElementById('view-' + tabId).classList.replace('hidden', 'block');
    document.getElementById('tab-' + tabId).classList.add('active');
}
```

### 3.2 即時天氣 API 整合 (Open-Meteo)
* **資料請求**：每天讀取對應城市的 `dayCoordinates` (緯度/經度)，動態向 API 發送請求。
* **快取機制**：使用全域 `weatherCache` 保存已請求的數據，避免多天切換時重複呼叫 API。
* **歷史數據備用**：當前日期大於系統預報極限（16天）或請求失敗時，靜態顯示 `itineraryData` 中的預設歷史均溫。

### 3.3 代辦清單存儲機制 (LocalStorage)
所有注意事項代辦清單項目透過 `toggleNotesCheck(id)` 進行狀態管理，並直接存入 `localStorage`，確保跨瀏覽器會話不會丟失狀態：
```javascript
function toggleNotesCheck(id) {
    const chk = document.getElementById('sep26_chk_' + id);
    const lbl = document.getElementById('sep26_lbl_' + id);
    if (chk && lbl) {
        localStorage.setItem('sep26_chk_' + id, chk.checked);
        if (chk.checked) {
            lbl.classList.add('line-through', 'text-gray-400');
        } else {
            lbl.classList.remove('line-through', 'text-gray-400');
        }
    }
}
```

---

## 4. 📊 資料結構與合約 (Data Structures & Contracts)

本節定義行程網頁中所使用之 JavaScript 全域資料變數與 JSON 格式規格。

### 4.1 行程事件格式 `itineraryData`

```javascript
const itineraryData = [
  {
    date: "YYYY-MM-DD",
    title: "當天主標題",
    tag: "城市/地區標籤",
    weather: "⛅ 歷史均溫",  // API 讀取失敗時的備用靜態氣溫
    events: [
      {
        time: "HH:MM",
        type: "sight",     // sight (景點) / food (餐飲) / buy (購物) / booking (預訂) / transport (交通) / flight (航班) / hotel (住宿) / story (故事)
        icon: "fas fa-...", // FontAwesome 圖示類名
        title: "事件名稱",
        desc: "詳細內容與建議遊玩時間",
        badges: [
          { cls: "sight", icon: "fa-ticket-alt", text: "標籤文字" }
        ],
        // 選用：警告/注意事項提醒框
        notice: {
          level: "danger", // danger (紅色字，特別重要) / warning (黃色字，一般警告)
          icon: "fa-exclamation-triangle",
          text: "提示標題",
          warning: "具體提醒細節"
        },
        // 選用：展開式藍色攻略框
        tip: {
          title: "攻略標題 🗺️",
          content: "HTML格式的文字（支援換行與粗體）"
        }
      }
    ]
  }
];
```

### 4.2 購物清單格式 `shoppingList`

```javascript
const shoppingList = [
  {
    category: "💊 分類名稱",
    color: "pink",          // 顏色鍵：pink / amber / purple / blue / green
    items: []               // 預設為空，由使用者在前端動態新增
  }
];
```

### 4.3 行李清單格式 `luggageList`

```javascript
const luggageList = [
  {
    category: "📄 分類名稱",
    color: "blue",          // 顏色鍵：blue / green / orange / red / purple / cyan
    items: [                // 純字串陣列
      "護照",
      "保暖外套"
    ]
  }
];
```

### 4.4 注意事項代辦清單格式 `notesChecklistData`

```javascript
const notesChecklistData = [
  {
    category: "🔴 超緊急（立刻辦！）",
    color: "red",            // 顏色鍵：red / orange / amber / blue
    items: [
      {
        id: "unique-id-01",  // 唯一辨識碼，用於 localStorage 保存狀態，不可重複
        text: "辦理日文駕照譯本",
        urgent: true         // 若為 true 則顯示「緊急」Badge
      }
    ]
  }
];
```

### 4.5 天氣城市座標對應 `dayCoordinates`

```javascript
const dayCoordinates = {
  0: {                       // key = 第幾天（索引從 0 開始）
    name: "別府",
    lat: 33.2830,
    lon: 131.4910,
    tenkiUrl: "https://tenki.jp/forecast/9/43/8220/44202/" // 點擊天氣跳轉的連結
  }
};
```

---

## 5. 📝 部署與自動化工作流

本專案採 GitHub Pages 靜態網站代管，任何程式碼更新或行程變更，請依循以下部署方式：

### 5.1 Git 手動部署
```powershell
git add .
git commit -m "feat(itinerary): 異動內容描述"
git push
```
GitHub Action 會自動觸發部署，約 **1-2 分鐘** 內生效。

### 5.2 Python 行程更新腳本 (`replace_itinerary.py`)
若行程資料過於龐大，可使用同目錄下的 Python 腳本，抽離 `itineraryData` 至獨立 json/js 檔案後，自動替換並推送到遠端倉庫。
