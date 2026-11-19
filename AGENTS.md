# 路徑規劃工具的 Codex 工作指引

## 開始工作

- 從 `O:\CHATGPT\route_plan\` 開啟專案，先讀 [PROJECT_HANDOFF.md](PROJECT_HANDOFF.md) 與 [CODEX_HANDOFF.md](CODEX_HANDOFF.md)，再查看根目錄 [index.html](index.html)。目前本機版本為 V1.5.5；來源基準為 V1.5.4，取自 `squall021/squall021.github.io` 的提交 `0d8fbfa255edd06398fa845803630e0d119751b8`。
- HTML、CSS、原生 JavaScript 都在根目錄 `index.html`。以 UTF-8 維護繁體中文介面。需要改版時，核對頁面 `<title>`、上限提示及 `data().version`。分享 payload 的 `v` 是資料格式版本，只有格式變更時才需要評估是否調整。
- 儲存庫為 [`squall021/route-planner`](https://github.com/squall021/route-planner)，Pages 從 `main` 分支根目錄發布至 [路徑規劃工具](https://squall021.github.io/route-planner/)。工具箱首頁及原工具資料夾由 `squall021.github.io` 儲存庫管理。

## 維護行為與相容性

- 目的地依手動順序編號。`S.v[i]` 存放第 `i`、`i+1` 個目的地之間的橘色定位點。目的地加上定位點合計最多 50 個，至少兩個目的地才呼叫 OSRM。目的地換序會在確認後清空定位點，刪除中間目的地會接合相鄰兩段定位點。留意 Leaflet `[lat, lng]` 與 OSRM／GeoJSON／KML `[lng, lat]` 的座標順序。
- `#share=` 是網址內的路線快照。只有 `e === 1` 開啟可編輯模式，缺少 `e` 的舊連結為唯讀。唯讀須隱藏編輯介面並停用標記拖曳。可編輯接收者的更動留在自己的工作狀態，可另存 JSON 或重新分享。保留 `g.`、`r.` 與舊無前綴 token 的讀取相容性。名稱等外部資料輸出到 HTML、XML 時使用適當的跳脫。
- 唯讀分享在 `820px` 以下（含）預設收合底部路線抽屜。展開時隱藏摘要，選站後收合、移動地圖並打開該站的標記。桌機保留左上清單。清單只列目的地。
- 維護 CSV／JSON 匯入與匯出相容性。GPX／KML 提供匯出。變更匯入驗證時，先完成驗證再替換目前路線。測試與備份使用範例座標，避開真實私人行程。
- Leaflet 與 QR Code 函式庫由 jsDelivr 載入。維持 OSM 地圖署名、瀏覽器 Referer 及正常快取。Nominatim 只用於使用者主動搜尋，公開服務的每秒 1 次上限涵蓋整個應用流量，禁止 autocomplete。OSRM 示範端點每秒最多 1 次，沒有可用性保證。自動化測試使用模擬回應。[Nominatim 使用政策](https://operations.osmfoundation.org/policies/nominatim/)、[OSM 圖磚政策](https://operations.osmfoundation.org/policies/tiles/)、[OSRM 示範服務政策](https://github.com/Project-OSRM/osrm-backend/wiki/Demo-server)

## 預覽、驗收與部署

1. 在專案根目錄執行 `python -m http.server 8000 --bind 127.0.0.1`，開啟 `http://127.0.0.1:8000/`。本機連結依目前網址產生。測試剪貼簿與瀏覽器定位時，使用 localhost 或 HTTPS 並確認權限。
2. 依 [交接說明的測試清單](CODEX_HANDOFF.md#測試清單) 驗收修改所涉及的功能，並檢查瀏覽器主控台。涉及分享時，同時驗證唯讀、可編輯、舊連結及 820／821px 兩側的行為。
3. 本機為獨立儲存庫工作樹，`origin` 必須為 `https://github.com/squall021/route-planner.git`。依使用者的發布授權提交及推送 `main`，待 Pages 部署完成後驗收正式工具。部署流程見 [CODEX_HANDOFF.md](CODEX_HANDOFF.md#日後部署)。原 `squall021.github.io` 的工具資料夾與工具箱首頁由使用者另行授權維護。


## Modern Web Development Guidance

- When modifying HTML, CSS, JavaScript, or browser APIs, consult the installed `modern-web-guidance` skill when relevant.
- Prefer modern, stable browser APIs over legacy implementations when they meet project requirements.
- Check browser compatibility before adopting newer web features.
- Apply progressive enhancement and lightweight fallbacks when necessary.
- Avoid adding unnecessary dependencies or large polyfills.
- Preserve existing behavior, layout, print dimensions, and export formats.
- Do not rewrite working code solely to adopt newer syntax.
- Prioritize performance, accessibility, and maintainability.
- For Chrome extension projects, consult the `chrome-extensions` skill and follow Manifest V3 guidance.
- Explain significant compatibility or architectural changes before implementing them.
- Do not automatically perform large-scale refactoring without explicit approval.

## 專案開發紀錄與交接規範

- 完成重要新功能、核心邏輯變更、重大 Bug 修正或架構調整後，依實際影響精簡更新下列文件。文案或版面等微調可略過本節的紀錄。
  - `PROJECT_HANDOFF.md` 記錄專案現況、待辦事項及已知問題。 詳細交接沿用 `CODEX_HANDOFF.md`，由本檔所列的接手摘要連結。
  - `DECISIONS.md` 記錄重要技術決策、採用理由及影響。
  - `CHANGELOG.md` 記錄重大功能變更及修正。
- 只編輯受影響的段落，以實際程式、設定及驗證結果為準，並標明尚未驗證的事項。文件不得記錄密碼、API 金鑰、權杖、憑證或其他秘密。
- 維護上述三份繁體中文 Markdown。缺少且確有需要時，依程式、版本紀錄與可查證的測試建立基線，證據不足時標明待確認。既有詳細交接或驗收紀錄以連結導覽，只讀及修改與任務相關的段落，避免重複記錄、重寫全文與浪費 Token。
