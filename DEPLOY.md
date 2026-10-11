# 部署到 GitHub Pages → trip.aosora.photography

Repo：`https://github.com/COYADI/sora-trip-designer`

## 上傳（網頁）
1. 開啟 https://github.com/COYADI/sora-trip-designer/upload
2. 把本機資料夾 `aosora-trip` 裡的檔案全部拖進去：
   - `index.html`、`CNAME`、`.nojekyll`、`iceland-2026/`（整個資料夾）
3. Commit changes

## GitHub Pages
1. https://github.com/COYADI/sora-trip-designer/settings/pages
2. Source：Deploy from a branch → `main` / `/ (root)` → Save
3. Custom domain：`trip.aosora.photography` → Save → 之後勾 Enforce HTTPS

## GoDaddy（繁中）
- 類型：CNAME
- 名稱：trip
- 內容值：`COYADI.github.io`
- 不要改 `@`、`www`