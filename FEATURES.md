# 旅遊行程網頁 - 功能需求記錄

> 記錄所有旅程行程規劃網頁的功能演進、已實現需求與待辦功能。
> 最後更新：2026-07-06

---

## 📁 專案列表

| 專案 | Repo | GitHub Pages | 描述 |
|------|------|-------------|------|
| 美西 16 日自由行 | — | [2026-la.vercel.app](https://2026-la.vercel.app/) | 原始參考版本 |
| 日本 10 日自由行 | [japan-trip](https://github.com/marksu1122/japan-trip) | [marksu1122.github.io/japan-trip](https://marksu1122.github.io/japan-trip/) | 6月版本 |
| 北九州 9 日自駕 | [japan-trip-sep-2026](https://github.com/marksu1122/japan-trip-sep-2026) | [marksu1122.github.io/japan-trip-sep-2026](https://marksu1122.github.io/japan-trip-sep-2026/) | 9月版本（本 repo） |

---

## ✅ 已實現功能（來自舊對話記錄）

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
- [x] `notice` 紅色警告框（重要提醒、地雷避坑）
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
- [x] 每項可打勾確認
- [x] 附數量欄位

### 🧳 行李清單
- [x] 分類清單（證件 / 金錢 / 衣物 / 藥品 / 3C / 盥洗）
- [x] 每項可打勾確認
- [x] 依旅程季節客製化（6月梅雨季 / 9-10月初秋）

### ✅ 注意事項代辦清單
- [x] 分階段提醒（緊急 / 出發前 1-2 個月 / 出發前 1-2 週 / 機上）
- [x] **勾選後自動儲存進度** (`localStorage`)，重開網頁不會消失
- [x] 「緊急」紅色 Badge 標記

### 🚗 自駕頁面
- [x] 每日自駕路線概覽
- [x] 重點路段資訊（高速公路名稱、車程、收費）
- [x] 警告框（塞車時段、停車建議、路況注意）

### 🏨 訂單資訊頁
- [x] 航班資訊（去程 / 回程）
- [x] 住宿清單（含連結、訂單號）
- [x] 租車資訊（取還車地點、車型、費用）

### 🔧 技術 / 維護
- [x] GitHub 版本控管
- [x] GitHub Pages 自動部署（push 後 1-2 分鐘生效）
- [x] `replace_itinerary.py` Python 腳本方便更新行程資料

---

## 🔄 各旅程客製化差異

| 功能 | 日本 6 月版 | 北九州 9 月版 |
|------|-----------|------------|
| 主題色 | Sakura 粉紅 `#f43f5e` | Autumn 橘色 `#f97316` |
| 動畫 | 🌸 飄落櫻花 | 🍁🍂 飄落楓葉 |
| 人數 | 未特別標示 | **6 人同行**（含注意事項）|
| 行程天數 | 10 天 | 9 天 |
| 氣候提醒 | 6 月梅雨季 | 9-10 月初秋 |
| localStorage key | `notes_chk_` | `sep26_chk_` |

---

## 💡 未來可新增功能（Backlog）

- [ ] **預算計算器** — 各費用分類（交通 / 住宿 / 餐飲 / 購物），自動換算台幣
- [ ] **地圖整合** — 每日行程對應 Google Maps 路線連結
- [ ] **6人費用分攤計算器** — 自動計算共同費用每人金額
- [ ] **照片相簿** — 旅遊照片上傳展示
- [ ] **離線 PWA** — Service Worker 讓手機可以離線使用
- [ ] **多語言** — 日文 / 英文版介面切換
- [ ] **訂單掃 QR Code** — 掃描後自動帶入訂單資訊
- [ ] **主頁 Portal** — 所有旅程的統一入口頁面

---

## 🔗 更新流程

每次修改行程後，執行以下指令推送到 GitHub Pages：

```powershell
# 在 japan-trip-sep-2026 目錄下
git add .
git commit -m "update: 說明修改內容"
git push
```

或使用 Python 腳本更新行程資料：
```powershell
python replace_itinerary.py
git add .; git commit -m "update: 更新行程資料"; git push
```

GitHub Pages 約 **1-2 分鐘**後自動生效。若看到舊版，按 `Ctrl + Shift + R` 強制重新整理。
