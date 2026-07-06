# 旅遊行程規劃網頁 — 完整建置藍圖

> **用途**：此文件是完整的技術規格書。  
> 根據本文件，可以從零開始建立一個功能完整、設計精美的旅遊行程規劃單頁網頁（SPA）。  
> 不需要任何框架或後端，只需要一個 `index.html` 檔案。

---

## 1. 專案概覽

### 1.1 目標
一個**行動優先**的旅遊行程規劃網頁，讓旅伴可以：
- 瀏覽每日行程（含景點攻略、警告提醒）
- 查看訂單憑證（航班、住宿、租車）
- 管理購物清單、行李清單
- 勾選行前代辦事項（進度自動儲存）
- 查看自駕路線規劃

### 1.2 技術棧
```
語言：HTML5 + Vanilla JavaScript + CSS
樣式：Tailwind CSS (CDN)
圖示：FontAwesome 6.4.0 (CDN)
字體：Google Fonts (Inter + Noto Serif TC)
API：Open-Meteo（即時天氣預報，免費無需 API Key）
部署：GitHub Pages（push 後自動更新）
```

### 1.3 檔案結構
```
project-name/
├── index.html          ← 唯一的程式檔案，所有功能都在此
├── FEATURES.md         ← 功能需求記錄
├── BLUEPRINT.md        ← 本文件（建置藍圖）
└── replace_itinerary.py ← 選用：用 Python 腳本批次更新行程資料
```

---

## 2. 設計系統

### 2.1 主題色（依旅程調整）

每個旅程選一個主題色系：

| 旅程類型 | 主色 | 淺色背景 | Tab Active | CSS 變數 |
|---------|------|---------|-----------|---------|
| 春季日本（櫻花）| `#f43f5e` | `#fff0f5` | `#111827` | `--sakura` |
| 秋季日本（楓葉）| `#f97316` | `#fff7ed` | `#c2410c` | `--autumn` |
| 美西藍色系 | `#3b82f6` | `#eff6ff` | `#1d4ed8` | `--ocean` |
| 自訂 | 任意 HSL 色值 | 對應淺色 | 深色版本 | 自訂 |

### 2.2 背景漸層
```css
/* 春季版 */
background: linear-gradient(135deg, #fff0f5 0%, #ffffff 40%, #ffffff 60%, #f0f4ff 100%);

/* 秋季版 */
background: linear-gradient(135deg, #fff7ed 0%, #ffffff 40%, #ffffff 60%, #fdf4ff 100%);
```

### 2.3 字體
```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Noto+Serif+TC:wght@500;700;900&display=swap" rel="stylesheet">
```
- **主字體**：`Inter`（UI 文字）
- **標題字體**：`Noto Serif TC`（中文標題，class: `font-serif-tc`）

### 2.4 動畫效果

#### 飄落動畫（依季節）
```css
@keyframes fall {
    0%   { transform: translateY(-20px) rotate(0deg); opacity: 1; }
    100% { transform: translateY(100vh) rotate(360deg); opacity: 0; }
}
.leaf {
    position: fixed;
    top: -20px;
    font-size: 1.2rem;
    animation: fall linear infinite;
    pointer-events: none;
    z-index: 0;
    user-select: none;
}
```
HTML 放在 `<body>` 最上方：
```html
<!-- 春季：🌸 | 秋季：🍁🍂 | 夏季：🌿 | 冬季：❄️ -->
<span class="leaf" style="left:5%;animation-duration:9s;animation-delay:0s;">🍁</span>
<span class="leaf" style="left:20%;animation-duration:12s;animation-delay:2s;">🍂</span>
<!-- 建議 6-7 個，分散 left 位置 -->
```

### 2.5 Badge 標籤系統
```css
.badge-pill  { display:inline-flex; align-items:center; gap:3px; padding:3px 8px; border-radius:6px; font-size:0.7rem; font-weight:600; }
.badge-food    { background:#fef2f2; color:#dc2626; border:1px solid #fecaca; }
.badge-buy     { background:#fdf2f8; color:#db2777; border:1px solid #fbcfe8; }
.badge-booking { background:#f0fdf4; color:#16a34a; border:1px solid #bbf7d0; }
.badge-story   { background:#f8fafc; color:#475569; border:1px solid #e2e8f0; }
.badge-sight   { background:#eff6ff; color:#2563eb; border:1px solid #bfdbfe; }
.badge-tip     { background:#fefce8; color:#ca8a04; border:1px solid #fef08a; }
.badge-green   { background:#f0fdf4; color:#16a34a; border:1px solid #bbf7d0; }
```

### 2.6 陰影
```css
.soft-shadow { box-shadow: 0 10px 30px -10px rgba(0,0,0,0.08), 0 0 15px rgba(255,255,255,0.6); }
```

---

## 3. HTML 整體結構

```
<html>
  <head>
    [Meta tags + PWA meta + CDN links + <style>]
  </head>
  <body>
    [飄落動畫 span × 6-7]
    
    <header>
      [旅程標題 + 地點副標題]
      [Tab 導航列]
    </header>
    
    <!-- VIEW 1：行程 -->
    <div id="view-itinerary">
      [日期選擇器（橫向捲動 Pill）]
      [主要行程卡片（動態渲染）]
    </div>
    
    <!-- VIEW 2：訂單資訊 -->
    <div id="view-tools" class="hidden">
      [航班資訊]
      [住宿清單]
      [租車資訊]
      [電子憑證連結]
    </div>
    
    <!-- VIEW 3：自駕 -->
    <div id="view-transport" class="hidden">
      [每日路線概覽]
      [重點路段]
      [停車資訊]
    </div>
    
    <!-- VIEW 4：購物 -->
    <div id="view-shopping" class="hidden">
      [購物清單（動態渲染）]
      [必買伴手禮介紹]
    </div>
    
    <!-- VIEW 5：行李 -->
    <div id="view-luggage" class="hidden">
      [行李清單（動態渲染）]
      [氣候參考表]
    </div>
    
    <!-- VIEW 6：注意事項 -->
    <div id="view-notes" class="hidden">
      [代辦清單（勾選 + localStorage 儲存）]
      [其他注意事項區塊]
    </div>
    
    <script>
      [資料區] itineraryData / shoppingList / luggageList / notesChecklistData
      [邏輯區] switchTab / buildDateNav / changeDay / toggleTip / render 函式 / 初始化
    </script>
  </body>
</html>
```

---

## 4. 資料結構規格

### 4.1 行程資料 `itineraryData`

```javascript
const itineraryData = [
  {
    date: "2026-09-26",        // ISO 格式，用於日期 Pill 計算星期
    title: "抵達福岡 & 取車出發",  // 當天主標題
    tag: "啟程",                 // 顯示在日期左上的小標籤
    weather: "☀️ 27°C",         // 歷史均溫（API 載入前顯示）
    events: [
      {
        time: "14:00",          // 時間（字串，可以是「待填」）
        type: "flight",         // 類型（見下表）
        icon: "fas fa-plane",   // FontAwesome class
        title: "抵達福岡機場",
        desc: "過海關、提領行李，前往租車公司取車。",
        
        // 選用：彩色小標籤（可多個）
        badges: [
          { cls: "booking", icon: "fa-plane", text: "入境辦理" }
        ],
        
        // 選用：紅色警告框
        notice: {
          icon: "fa-clock",            // FontAwesome class（不含 fas）
          text: "標題文字",
          warning: "警告內容說明"
        },
        
        // 選用：藍色可展開攻略框
        tip: {
          title: "攻略標題 🗺️",
          content: "詳細內容（可包含 <b>HTML</b> 標籤）"
        }
      }
    ]
  }
];
```

#### Event Type 對照表
| type | 顏色 | 適用場景 |
|------|------|---------|
| `food` | 紅色系 | 餐廳、美食 |
| `sight` | 藍色系 | 景點、觀光 |
| `buy` | 粉色系 | 購物、市場 |
| `booking` | 綠色系 | 預訂確認、取車、Check-in |
| `transport` | 灰色系 | 交通移動、開車 |
| `flight` | 藍色系 | 航班 |
| `hotel` | 紫色系 | 住宿、溫泉旅館 |
| `story` | 灰色系 | 自由活動、其他 |

---

### 4.2 購物清單 `shoppingList`

```javascript
const shoppingList = [
  {
    category: "💊 藥妝保養",   // 分類標題（含 emoji）
    color: "pink",            // 顏色鍵：pink / amber / purple / blue / green
    items: [
      { name: "眼藥水（參天）", qty: "2 瓶" },  // qty 可省略
      { name: "痠痛貼布" }
    ]
  }
];
```

#### 可用 color 值
| color | 背景 | 邊框 | 標題色 |
|-------|------|------|-------|
| `pink` | bg-pink-50 | border-pink-100 | text-pink-800 |
| `amber` | bg-amber-50 | border-amber-100 | text-amber-800 |
| `purple` | bg-purple-50 | border-purple-100 | text-purple-800 |
| `blue` | bg-blue-50 | border-blue-100 | text-blue-800 |
| `green` | bg-green-50 | border-green-100 | text-green-800 |

---

### 4.3 行李清單 `luggageList`

```javascript
const luggageList = [
  {
    category: "📄 證件文件",
    color: "blue",           // 顏色鍵：blue / green / orange / red / purple / cyan
    items: [                 // 純字串陣列
      "護照（有效期 6 個月以上）",
      "日文駕照譯本（租車必須！）"
    ]
  }
];
```

---

### 4.4 注意事項代辦清單 `notesChecklistData`

```javascript
const notesChecklistData = [
  {
    category: "🔴 超緊急（立刻辦！）",
    color: "red",            // 顏色鍵：red / orange / amber / blue
    items: [
      {
        id: "unique-id-01",  // 唯一 ID（用於 localStorage key，不可重複）
        text: "辦理日文駕照譯本",
        urgent: true         // 顯示紅色「緊急」Badge
      },
      {
        id: "unique-id-02",
        text: "預訂租車"
        // urgent 省略 = false
      }
    ]
  }
];
```

> ⚠️ **重要**：`id` 在同一旅程的所有項目中必須唯一。  
> 建議命名規則：`{旅程縮寫}-{功能縮寫}`，例如 `sep26-lic-trans`

---

### 4.5 天氣座標 `dayCoordinates`

```javascript
const dayCoordinates = {
  0: {                                    // key = 第幾天（從 0 開始）
    name: "福岡",
    lat: 33.5902,
    lon: 130.4017,
    tenkiUrl: "https://tenki.jp/forecast/9/46/8210/40132/"  // 日本天氣網站連結
  },
  1: { name: "福岡", lat: 33.5902, lon: 130.4017, tenkiUrl: "..." },
  // 每天一個，與 itineraryData 索引對應
};
```

**常用城市座標速查**：
| 城市 | lat | lon |
|------|-----|-----|
| 福岡 | 33.5902 | 130.4017 |
| 長崎 | 32.7503 | 129.8779 |
| 佐世保 | 33.1802 | 129.7150 |
| 別府 | 33.2830 | 131.4910 |
| 由布院 | 33.2600 | 131.3590 |
| 阿蘇 | 32.8844 | 131.1044 |
| 熊本 | 32.7900 | 130.7417 |
| 東京 | 35.6762 | 139.6503 |
| 大阪 | 34.6937 | 135.5022 |
| 京都 | 35.0116 | 135.7681 |
| 下田 | 34.6796 | 138.9442 |
| 熱海 | 35.0962 | 139.0717 |
| 箱根 | 35.2324 | 139.1039 |

---

## 5. JavaScript 邏輯規格

### 5.1 Tab 切換 `switchTab(tabId)`
- Tab ID 對應：`itinerary` / `tools` / `transport` / `shopping` / `luggage` / `notes`
- 切換時：隱藏所有 view → 顯示目標 view
- 購物/行李/注意事項首次開啟時執行對應 render 函式

### 5.2 日期導航 `buildDateNav()`
- 讀取 `itineraryData`，為每天建立一個 Pill 按鈕
- Pill 顯示：星期縮寫（SUN/MON...）+ 日期數字
- Active Pill 套用主題深色背景

### 5.3 行程渲染 `changeDay(idx)`
完整邏輯：
1. 更新日期卡片（日期數字 / 星期 / 月份年份）
2. 顯示歷史均溫，同時呼叫 `updateWeatherRealtime` 取 API 資料
3. 清空事件容器，逐一建立事件卡片 DOM
4. 每張卡片包含：圖示 + 時間 + 類型標籤 + 標題 + 描述 + badges + notice + tip
5. 更新日期 Pill 的 active 狀態
6. 控制上一天/下一天按鈕的顯示
7. 執行淡入動畫（opacity 0 → 1，translateY 8px → 0）

### 5.4 天氣 API `updateWeatherRealtime(dayDate, idx)`
```
API: https://api.open-meteo.com/v1/forecast
參數: latitude, longitude, daily=weathercode,temperature_2m_max,temperature_2m_min
      timezone=Asia/Tokyo, forecast_days=16
```
- 有快取：`weatherCache[idx]` 存放結果，避免重複請求
- WMO 天氣碼對應 emoji（0=☀️, 1=🌤️, 2=⛅, 3=☁️, 51-65=🌧️, 95=⛈️）
- 天氣文字可點擊，連結到 tenki.jp 詳細頁

### 5.5 Tip 展開 `toggleTip(tipId)`
- 切換 `.tip-body` 的 `open` class
- 旋轉箭頭圖示（0° ↔ 180°）
- 使用 CSS `max-height` transition 製造平滑展開效果

### 5.6 注意事項勾選 `toggleNotesCheck(id)`
- 讀取對應 checkbox 狀態
- 寫入 `localStorage`（key: `{旅程前綴}_chk_{id}`）
- 切換文字的 `line-through` 樣式

### 5.7 初始化
```javascript
document.addEventListener('DOMContentLoaded', () => {
    buildDateNav();
    changeDay(0);
    
    // 若今天在旅程期間，自動跳到今天
    const today = new Date().toISOString().split('T')[0];
    const todayIdx = itineraryData.findIndex(d => d.date === today);
    if (todayIdx !== -1) changeDay(todayIdx);
    
    // 預先渲染注意事項（讓 localStorage 勾選狀態立即生效）
    renderNotesChecklist();
});
```

---

## 6. Meta Tags（PWA 支援）

```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="default">
<meta name="apple-mobile-web-app-title" content="旅程簡稱">
<meta name="mobile-web-app-capable" content="yes">
<meta name="theme-color" content="#fff7ed">  <!-- 主題淺色 -->
<meta name="description" content="旅程描述">
<link rel="shortcut icon" href="https://cdn-icons-png.flaticon.com/512/197/197604.png">
```

---

## 7. Header 結構

```html
<header class="pt-6 md:pt-10 pb-3 text-center px-3 relative z-10">
    <!-- 小標 -->
    <p class="text-xs tracking-[0.25em] text-{主題}-500 font-semibold uppercase mb-1">✈️ 旅遊行程規劃</p>
    
    <!-- 主標題 -->
    <h1 class="text-[1.45rem] md:text-[2.2rem] font-serif-tc font-bold tracking-[0.1em] text-[#1a1a1a] mb-1">
        北九州 9 日自駕
    </h1>
    
    <!-- 地點副標題 -->
    <p class="text-sm text-gray-400 mb-5">🗾 福岡 · 長崎 · 佐世保 · 別府 · 由布院</p>

    <!-- Tab 導航 -->
    <div class="max-w-5xl mx-auto overflow-x-auto no-scrollbar pb-3 px-1 text-center">
        <div class="inline-flex bg-white/60 backdrop-blur-md p-1 rounded-full shadow-sm border border-gray-200">
            <button onclick="switchTab('itinerary')" id="tab-itinerary" class="tab-btn active ...">
                <i class="fas fa-calendar-alt mr-1"></i>行程
            </button>
            <!-- 其他 Tab 按鈕... -->
        </div>
    </div>
</header>
```

Tab 按鈕的 active CSS：
```css
.tab-btn.active {
    background-color: #c2410c;   /* 主題深色 */
    color: white;
    box-shadow: 0 4px 12px rgba(194,65,12,0.3);
}
```

---

## 8. 主要行程卡片結構

```html
<div id="main-card-content" class="bg-white/95 backdrop-blur-xl rounded-[1.5rem] md:rounded-[2rem] soft-shadow max-w-3xl mx-auto p-4 sm:p-6 md:p-10 border border-white">
    
    <!-- 日期 + 標題列 -->
    <div class="flex flex-row items-center gap-3 md:gap-5 mb-5 md:mb-8">
        <!-- 日期方塊 -->
        <div class="shrink-0 flex flex-col items-center justify-center w-16 h-16 md:w-[5.5rem] md:h-[5.5rem] border border-{主題}-100 rounded-[1rem] bg-{主題}-50 shadow-sm">
            <span id="card-date">--</span>
            <span id="card-day-week">---</span>
            <span id="card-month">---</span>
        </div>
        <!-- 標題 -->
        <div class="flex-1 min-w-0">
            <span id="card-tag">城市</span>
            <div id="card-weather"></div>
            <h2 id="card-title">載入中...</h2>
        </div>
    </div>

    <!-- 事件列表（動態渲染）-->
    <div class="space-y-3 md:space-y-4" id="card-events"></div>

    <!-- 上一天/下一天 -->
    <div class="flex justify-between mt-8 pt-6 border-t border-gray-100">
        <button id="btn-prev" onclick="changeDay(currentIndex - 1)">← 上一天</button>
        <button id="btn-next" onclick="changeDay(currentIndex + 1)">下一天 →</button>
    </div>
</div>
```

---

## 9. 事件卡片渲染模板

每個事件卡片的 HTML 結構（JS 動態生成）：

```html
<div class="day-event-card {bg} {border} border rounded-xl p-3 md:p-4">
    <div class="flex gap-3">
        <!-- 圖示 -->
        <div class="shrink-0 w-9 h-9 rounded-full bg-white flex items-center justify-center {icon-color} shadow-sm border {border}">
            <i class="{ev.icon} text-sm"></i>
        </div>
        <!-- 內容 -->
        <div class="flex-1 min-w-0">
            <div class="flex flex-wrap items-center gap-2 mb-1">
                <span class="text-[11px] font-bold {time-color} font-mono">{ev.time}</span>
                <span class="badge-pill badge-{type}">類型標籤</span>
            </div>
            <p class="font-bold text-[13.5px] md:text-[14.5px] text-gray-800">{ev.title}</p>
            <p class="text-[12px] md:text-[13px] text-gray-500 mt-1">{ev.desc}</p>
        </div>
    </div>
    
    <!-- badges（若有）-->
    <div class="flex flex-wrap gap-1.5 mt-2">
        <span class="badge-pill badge-{cls}"><i class="fas {icon}"></i> {text}</span>
    </div>
    
    <!-- notice（若有）-->
    <div class="mt-2.5 bg-white/70 rounded-xl border border-gray-100 p-3">
        <div class="flex items-center gap-2 mb-1.5">
            <i class="fas {notice.icon} text-gray-400"></i>
            <span class="font-bold text-gray-600">{notice.text}</span>
        </div>
        <div class="flex items-start gap-2 bg-red-50 rounded-lg border border-red-100 p-2">
            <i class="fas fa-exclamation-triangle text-red-400"></i>
            <span class="text-red-700 font-medium">{notice.warning}</span>
        </div>
    </div>
    
    <!-- tip（若有，可展開）-->
    <div class="mt-2.5 bg-blue-50/60 rounded-xl border border-blue-100">
        <button onclick="toggleTip('{tipId}')">
            <i class="fas fa-info-circle"></i>
            {tip.title}
            <i class="fas fa-chevron-down"></i>
        </button>
        <div id="{tipId}" class="tip-body px-4">
            <div class="pb-4 text-gray-600">{tip.content}</div>
        </div>
    </div>
</div>
```

---

## 10. 訂單資訊頁建議結構

包含以下區塊（每個區塊都是白色卡片）：

1. **航班資訊**：去程 + 回程，各包含航班號、日期、時間、航廈、行李額度
2. **住宿清單**：每間飯店一列，包含名稱、入住日期、訂單號、憑證連結
3. **租車資訊**：車型、取/還車時間地點、ETC、保險、費用

```html
<!-- 卡片外層 -->
<div class="bg-white/95 backdrop-blur-xl rounded-[1.5rem] soft-shadow p-5 md:p-6 border border-white">
    <h3 class="text-[1.1rem] md:text-lg font-bold text-gray-800 mb-3 border-b pb-2 flex items-center gap-2">
        <i class="fas fa-plane text-blue-500"></i> 航班資訊
    </h3>
    <!-- 內容 -->
</div>
```

---

## 11. GitHub 部署流程

### 首次建立
```powershell
# 1. 進入專案目錄
cd C:\path\to\project-name

# 2. 初始化 Git
git init
git config user.email "marksu1122@users.noreply.github.com"
git config user.name "marksu1122"

# 3. 加入並提交
git add .
git commit -m "feat: Initial commit - {旅程名稱}"

# 4. 建立 GitHub repo 並推送（需已登入 gh CLI）
gh repo create {repo-name} --public --source=. --remote=origin --push --description "{描述}"

# 5. 啟用 GitHub Pages
$body = '{"source":{"branch":"master","path":"/"}}'; $body | gh api repos/marksu1122/{repo-name}/pages --method POST --input -
```

### 後續更新
```powershell
git add .
git commit -m "update: {說明}"
git push
# GitHub Pages 約 1-2 分鐘後自動更新
```

### 網址格式
- **Repo**：`https://github.com/marksu1122/{repo-name}`
- **網頁**：`https://marksu1122.github.io/{repo-name}/`

---

## 12. 新旅程建立 Checklist

複製以下清單，建立新旅程時逐項完成：

```
□ 1. 確定旅程名稱 → repo 命名規則：{地點}-trip-{月份}-{年份}
□ 2. 選擇主題色（春/秋/夏/冬/自訂）
□ 3. 複製現有 index.html 作為範本
□ 4. 修改 <title> 和 meta 標籤
□ 5. 修改 Header 標題、副標題、地點
□ 6. 替換飄落動畫 emoji 和顏色
□ 7. 修改 Tab active 顏色（CSS .tab-btn.active）
□ 8. 修改 .date-pill.active 顏色
□ 9. 修改 card-date 區塊 border/bg 顏色
□ 10. 填入 itineraryData（每天的行程）
□ 11. 填入 dayCoordinates（每天的天氣座標）
□ 12. 填入 shoppingList（購物清單）
□ 13. 填入 luggageList（行李清單）
□ 14. 填入 notesChecklistData（代辦清單，注意 ID 唯一性）
□ 15. 更新訂單資訊頁內容（航班/住宿/租車）
□ 16. 更新自駕頁路線概覽
□ 17. 確認 localStorage key 前綴唯一（避免跨旅程衝突）
□ 18. 在瀏覽器本機測試（直接開啟 index.html）
□ 19. git init + commit + gh repo create
□ 20. 啟用 GitHub Pages
□ 21. 確認網頁可正常訪問
□ 22. 更新 FEATURES.md 的專案列表
```

---

## 13. 常見問題 & 解法

| 問題 | 原因 | 解法 |
|------|------|------|
| GitHub Pages 還是舊版 | 瀏覽器快取 | `Ctrl+Shift+R` 強制重新整理，或用無痕視窗 |
| 天氣 API 沒有資料 | 日期超出 16 天預報範圍 | 顯示「歷史均溫」是正常的 |
| 勾選清單重開後消失 | localStorage key 衝突 | 確保每個旅程的 `id` 前綴不同 |
| 手機版 Tab 文字太擠 | 螢幕太窄 | Tab 列已設定橫向捲動（`overflow-x-auto no-scrollbar`）|
| Tip 展開動畫卡頓 | `max-height` transition 設定問題 | 確保 `.tip-body` 有 `transition: max-height 0.4s` |

---

## 14. 版本記錄

| 日期 | 版本 | 說明 |
|------|------|------|
| 2026-05 | v1.0 | 日本 10 日（6月），參考美西版建立 |
| 2026-06 | v1.1 | 加入即時天氣 API、導遊攻略 tips |
| 2026-07 | v2.0 | 北九州 9 日（9月），秋季主題，6人版本 |
| 2026-07 | v2.1 | 新增景點 SOP 標準流程（第 15 節）|

---

## 15. 新增景點 SOP（標準作業流程）

> 當使用者說「幫我加這個景點」時，AI 必須依照此流程逐步執行。  
> **不可跳過任何步驟直接寫入程式碼。**

---

### 📋 Phase 1：資訊收集（問使用者）

收到景點名稱後，**先確認以下資訊再動手**：

```
必問項目：
□ A. 景點要放在哪一天？（哪個日期）
□ B. 大概幾點去？（早上 / 下午 / 晚上，或指定時間）
□ C. 預計停留多久？
□ D. 這個景點取代原本行程中的哪個活動，還是插入在哪個活動之間？
```

> 如果使用者直接說「放第 3 天下午」這類資訊，A/B 可以不用再問。  
> 如果全天行程已排滿，需提示使用者確認是否調整其他行程。

---

### 🔍 Phase 2：景點資料研究（AI 自行查詢）

確認放哪天之後，**使用 `search_web` 查詢以下資訊**：

```
必查項目：
□ 1. 正式名稱（含日文原名）
□ 2. 營業時間（平日 vs 週末是否不同？）
□ 3. 公休日（通常是週幾？節假日是否營業？）
□ 4. 門票價格（成人 / 兒童 / 是否有聯票優惠）
□ 5. 預約制 vs 現場排隊？（是否需要事先訂票）
□ 6. 排隊等候時間（旺季平均幾分鐘？）
□ 7. 從當天前一個景點過去的車程 / 交通方式
□ 8. 停車資訊（若是自駕旅程）
□ 9. 最佳參觀時段（避開人潮的技巧）
□ 10. 該景點有沒有特別注意事項（入場規定、禁止拍照、服裝要求等）
```

> 若景點在旅程日期之後超過 16 天，天氣 API 無法預報，tip 中補充該地區季節特性即可。

---

### ⏱️ Phase 3：時間軸衝突檢查

在插入景點前，**檢查當天已有的行程**：

```
檢查項目：
□ 前一個活動的結束時間 + 交通時間 ≤ 新景點開始時間？
□ 新景點結束時間 + 交通時間 ≤ 下一個活動開始時間？
□ 景點營業時間是否涵蓋預計到訪時段？
□ 若是週末，是否有週末限定的排隊或人潮問題？
□ 6人同行的特殊考量（訂位需提前、車位數量、集體票等）
```

若有衝突，**先向使用者說明**，提供調整建議，等確認後再寫入。

---

### ✍️ Phase 4：寫入行程資料

確認無衝突後，依照以下規則寫入 `itineraryData`：

#### 4.1 事件物件範本
```javascript
{
    time: "14:00",           // 根據研究結果填入建議時間
    type: "sight",           // 景點用 sight，餐廳用 food，購物用 buy
    icon: "fas fa-mountain", // 選擇最貼切的 FontAwesome icon
    title: "景點名稱（日文原名）",
    desc: "一句話說明景點特色 + 建議停留時間。",
    badges: [
        { cls: "sight", icon: "fa-ticket-alt", text: "¥門票金額" },
        { cls: "booking", icon: "fa-clock", text: "所需時間" }
    ],
    // 若有排隊 / 預約 / 特殊限制 → 用 notice
    notice: {
        icon: "fa-clock",
        text: "排隊 / 注意事項標題",
        warning: "具體說明：平均排隊時間、建議幾點前到、是否需提前預訂"
    },
    // 景點攻略細節 → 用 tip（可展開）
    tip: {
        title: "景點攻略 🗺️",
        content: "1. 最佳入場時間：XX:XX 前（避開人潮）<br>2. 必看重點：XXX<br>3. 停車：XX 停車場，距離 X 分鐘步行<br>4. 購票：現場 / Klook / 官網"
    }
}
```

#### 4.2 icon 選擇速查
| 景點類型 | FontAwesome icon |
|---------|----------------|
| 山岳 / 火山 | `fa-mountain` |
| 神社 / 寺廟 | `fa-torii-gate` / `fa-place-of-worship` |
| 城堡 | `fa-chess-rook` |
| 博物館 | `fa-museum` |
| 主題樂園 | `fa-ferris-wheel` |
| 溫泉 | `fa-hot-tub` |
| 海灘 / 湖泊 | `fa-water` |
| 購物 | `fa-shopping-bag` |
| 餐廳 / 美食 | `fa-utensils` |
| 夜景 / 展望台 | `fa-binoculars` |
| 橋梁 / 建築 | `fa-archway` |
| 自然步道 | `fa-leaf` |
| 動物園 / 水族館 | `fa-paw` |
| 港口 / 船隻 | `fa-ship` |

#### 4.3 tip 內容必寫項目
```
tip.content 必須包含：
1. 建議到訪時間（幾點最好 / 避開人潮）
2. 門票購買方式（現場 / 線上預購 / Klook）
3. 停車資訊（若自駕旅程）
4. 1-3 個當地獨家提醒或必看重點
```

---

### 🚀 Phase 5：更新並部署

```powershell
git add .
git commit -m "feat(itinerary): 新增 {景點名稱} @ Day {天數} {日期}"
git push
```

GitHub Pages 約 **1-2 分鐘**後生效。

---

### 📝 新增景點 快速對話範例

使用者說：「幫我加阿蘇火山那邊的草千里濱」

AI 應回應：
```
好的！草千里濱確認幾件事：
1. 要放在哪一天？（目前 Day 7 / 10/2 有阿蘇山行程）
2. 是要加在火口展望台之前還是之後？
3. 預計停留多久？（草千里通常 30-60 分鐘）
```

使用者回：「Day 7 火口之後，停 45 分」

AI 接著：
1. 查詢草千里濱資訊（免費、全天開放、停車場 ¥500）
2. 確認時間：火口結束約 11:30 → 草千里 11:45 → 結束約 12:30，不衝突
3. 寫入 event 物件，加入 tip（秋天高原牛隻放牧、絕佳拍照角度）
4. commit + push

---

### ⚠️ SOP 禁止事項

```
✗ 不可不查資訊就直接寫入（可能營業時間有誤）
✗ 不可不確認插入位置就修改行程（可能造成時間衝突）
✗ tip 不可只寫一行（至少要有 3 個實用資訊）
✗ 不可忘記 commit message 格式：feat(itinerary): 新增 {景點} @ Day {N} {日期}
```
