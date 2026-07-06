# 🗺️ Trip Planner Template

> 旅遊行程規劃網頁通用範本。  
> 從這個 repo clone，快速建立任何旅程的行程規劃網站。

## 📂 文件結構

| 檔案 | 說明 |
|------|------|
| `BLUEPRINT.md` | ⭐ **完整技術規格書**，包含所有設計決策、資料結構、邏輯說明 |
| `index.template.html` | 空白起始範本，複製後填入旅程資料即可 |
| `FEATURES.md` | 所有功能需求記錄（已實現 / Backlog）|

## 🚀 快速開始

```bash
# 1. Clone 這個 template
gh repo clone marksu1122/trip-planner-template my-new-trip

# 2. 複製範本
cp index.template.html index.html

# 3. 編輯 index.html，填入旅程資料
#    依照 BLUEPRINT.md 第 12 節的 Checklist 逐項完成

# 4. 建立新 repo 並部署
git init
git add .
git commit -m "feat: Initial commit"
gh repo create {trip-name} --public --source=. --remote=origin --push
$body = '{"source":{"branch":"master","path":"/"}}'; $body | gh api repos/marksu1122/{trip-name}/pages --method POST --input -
```

## 📋 已建立的旅程

| 旅程 | Repo | 網頁 |
|------|------|------|
| 🇯🇵 日本 10 日自由行（2026/06） | [japan-trip](https://github.com/marksu1122/japan-trip) | [🔗 開啟](https://marksu1122.github.io/japan-trip/) |
| 🇯🇵 北九州 9 日自駕（2026/09） | [japan-trip-sep-2026](https://github.com/marksu1122/japan-trip-sep-2026) | [🔗 開啟](https://marksu1122.github.io/japan-trip-sep-2026/) |

## 📖 相關文件

- 詳細建置流程 → [`BLUEPRINT.md`](./BLUEPRINT.md)
- 功能需求記錄 → [`FEATURES.md`](./FEATURES.md)
