# 路徑規劃工具

目前版本 **V1.5.5**。使用 Leaflet／OpenStreetMap 顯示地圖、OSRM 規劃汽車路線，並以 Nominatim 提供使用者主動搜尋。

- 儲存庫：[squall021/route-planner](https://github.com/squall021/route-planner)
- GitHub Pages 網址：[路徑規劃工具](https://squall021.github.io/route-planner/)
- 網站入口為根目錄 `index.html`，Pages 從 `main` 分支的 `/` 發布。

## 本機預覽

```powershell
Set-Location -LiteralPath 'O:\CHATGPT\route_plan'
python -m http.server 8000 --bind 127.0.0.1
```

開啟 <http://127.0.0.1:8000/>。網站直接由瀏覽器執行，互動預覽需要連線至 CDN 與地圖服務。自動化驗收使用範例資料及模擬回應。

## 功能

- 目的地改名、複製、換序、移除及拖曳，支援分路段定位點，合計最多 50 點。
- 唯讀／可編輯分享網址及 QR Code，手機可收合路線清單。
- CSV／JSON 匯入匯出、GPX／KML 匯出、瀏覽器列印與手動儲存／讀取。

## 維護文件

[接手摘要](PROJECT_HANDOFF.md) 包含目前狀態、驗收與已知問題；[詳細交接](CODEX_HANDOFF.md) 提供資料模型與格式說明。[技術決策](DECISIONS.md)、[變更紀錄](CHANGELOG.md) 及 [開發指引](AGENTS.md) 隨專案維護。

本專案由 `squall021/squall021.github.io` 的 `0d8fbfa255edd06398fa845803630e0d119751b8` 來源快照移出，保留本機新增的 V1.5.5 功能。原儲存庫的工具資料夾與工具箱首頁繼續由原儲存庫管理。
