# 路徑規劃工具｜Codex 詳細交接

> 2026-11-19 移至獨立儲存庫 [squall021/route-planner](https://github.com/squall021/route-planner)，目前 V1.5.5，入口為根目錄 [index.html](index.html)。目前狀態與已知問題見 [PROJECT_HANDOFF.md](PROJECT_HANDOFF.md)。下列來源表記錄 V1.5.4 的交接基準。

## 來源與本機結構

| 項目 | 交接基準 |
| --- | --- |
| 核對日期 | 2026-11-13（台灣時間） |
| 上游 | [`squall021/squall021.github.io`](https://github.com/squall021/squall021.github.io)，預設分支 `main` |
| 來源快照 | [`0d8fbfa255edd06398fa845803630e0d119751b8`](https://github.com/squall021/squall021.github.io/commit/0d8fbfa255edd06398fa845803630e0d119751b8)（2026-11-03） |
| 最近一次修改工具 | [`60b4ee0ab42aa383a5e8b8b3e87fbe3fef5555a7`](https://github.com/squall021/squall021.github.io/commit/60b4ee0ab42aa383a5e8b8b3e87fbe3fef5555a7)（2026-09-13，V1.5.4 手機抽屜） |
| 工具原始碼 | [`route-planner/index.html`](https://github.com/squall021/squall021.github.io/blob/0d8fbfa255edd06398fa845803630e0d119751b8/route-planner/index.html)，39,515 bytes，SHA-256 `64e565c41287905f8209005375496a01bba7948f4be67e53480ec12469f7b149` |
| 正式入口 | [路徑規劃工具](https://squall021.github.io/route-planner/) |
| 工具箱首頁 | [我的工具箱](https://squall021.github.io/) 目前有 9 張卡片。路徑規劃卡片顯示 `V1.5.4`，連至 `https://squall021.github.io/route-planner/`。卡片寫在[上游根目錄 `index.html`](https://github.com/squall021/squall021.github.io/blob/0d8fbfa255edd06398fa845803630e0d119751b8/index.html) |

本機以 `O:\CHATGPT\route_plan\` 為工作根目錄。2026-11-19 將目前 V1.5.5 入口搬至根目錄並建立獨立 Git 工作樹，唯一 `origin` 為 `https://github.com/squall021/route-planner.git`。原始儲存庫的 Git 歷史、工具箱首頁、其他工具及工作流程繼續由原儲存庫管理。2026-11-13 查核時兩個正式網址皆回傳 HTTP 200，HTML 分別與上述提交的 Git blob 逐位元組一致。

```text
O:\CHATGPT\route_plan\
├── .gitignore
├── .nojekyll
├── AGENTS.md
├── CHANGELOG.md
├── CODEX_HANDOFF.md
├── DECISIONS.md
├── PROJECT_HANDOFF.md
├── README.md
└── index.html
```

## 架構與功能入口

- 單一靜態頁面包含 HTML、CSS 和原生 JavaScript（IIFE）。繁體中文介面直接由瀏覽器執行，無套件安裝、建置步驟、後端、資料庫或 Google API Key。jsDelivr 載入 Leaflet `1.9.4` 和 qrcodejs `1.0.0`。初始地圖為台灣 `[23.7, 121]`、縮放 7。
- `E` 集中 DOM 參照，`S` 維護路線狀態。`render()` 更新編輯清單、路段選單與 Leaflet 標記。`renderMarkers()` 更新可拖曳狀態、標記及其資訊視窗。`fit()` 依路線點數調整地圖視野。
- `search()` 由搜尋按鈕或 Enter 觸發，可找地名／地址，也可輸入 `25.0330, 121.5654` 直接定位。搜尋結果支援查看及加入點位。可在地圖上點選加入目的地，或用瀏覽器「目前位置」加入目的地。目的地可以改名、刪除、上下換序及拖曳。
- 路線定位點屬於選定的路段，可從搜尋結果或地圖加入，支援拖曳、上下換序、刪除與清除整段。目的地顯示數字，定位點顯示橘色。目的地換序前會確認清除既有定位點。刪除中間目的地會串接相鄰路段的定位點，刪除首尾目的地會移除對應端的路段。
- `ordered()` 將目的地與每一段定位點交錯展開為行駛順序。`route()` 按這個順序請求 OSRM `route/v1/driving`，使用 `overview=full&geometries=geojson&steps=true`，取第一條結果畫線並更新距離與預估時間。至少要有兩個目的地，總點數上限是 50 個（目的地加定位點）。現行功能為按手動順序規劃汽車路線。
- 手動「儲存／讀取」使用瀏覽器 `localStorage` 的 `route-planner-v151` 單一槽位。搜尋快取用 `route-search-cache`。儲存與快取依瀏覽器設定檔及來源（協定、主機、連接埠）隔離。正式頁的瀏覽器資料不隨原始碼搬到本機，可用 JSON 匯出再匯入。

### 資料模型與座標

| 欄位 | 說明 |
| --- | --- |
| `S.d` | 有序目的地陣列 |
| `S.v` | 以相鄰目的地間的路段為索引。`S.v[i]` 屬於 `S.d[i] → S.d[i+1]`，有 `max(0, S.d.length - 1)` 個子陣列 |
| 點位 | `{ id, type, name, lat, lng, displayName }`。`type` 為 `destination` 或 `via`。緯度 `-90..90`、經度 `-180..180`，數值必須有限 |
| `S.geo`、`S.distance`、`S.duration` | OSRM GeoJSON 路線幾何、距離（公尺）、時間（秒） |
| 座標順序 | 手動輸入與 Leaflet 使用緯度、經度。OSRM 請求、GeoJSON 與 KML 使用經度、緯度。GPX 使用 `lat`、`lon` 屬性 |

## V1.5.4 分享與手機抽屜

1. 透過上方「分享」或側欄「產生分享連結」操作，至少需要一個目的地。初次開啟頁面時「允許收到此連結的人編輯路線」未勾選。分享視窗關閉再開啟時會沿用目前勾選狀態。
2. `sharePayload()` 產生 `{v:2,e,n,c,u,d,w}`。`v` 是分享格式版本，`e` 為 `0` 或 `1`，`n` 為路線名稱，`c`、`u` 為建立及產生分享的 ISO 時間。`d` 是目的地、`w` 是按路段分組的定位點。點位為 `[name, lat, lng]`，座標保留到小數點後六位。接收端用快照重建點位並重新請求路線，因此距離和時間可能隨路網更新。
3. `packShare()` 把 JSON 轉為 UTF-8；支援 `CompressionStream` 時以 gzip 壓縮並使用 `g.` 前綴，其他瀏覽器用未壓縮的 `r.`。兩者都用去除填充的 Base64url，解碼也相容舊的無前綴原始資料。`g.` 需要 `DecompressionStream`，可另外提供 JSON 備份給較舊瀏覽器。
4. URL 由 `location.origin + location.pathname + '#share=' + token` 產生。以本機頁面產生的網址會指向本機。長度不超過 1,800 字元且 qrcodejs 成功載入時才嘗試畫 QR Code（190 × 190、容錯 M）。過長可複製網址或提供 JSON 檔，並測試收件的通訊軟體是否截斷網址。1800 是本頁的產生門檻。
5. `initSharedRoute()` 以 `p.e === 1` 判斷可編輯。唯讀連結隱藏工具列與側欄、禁止標記拖曳及地圖加點，仍可瀏覽地圖、摘要、路線名稱、時間及目的地清單。舊連結缺少 `e` 時以唯讀開啟。可編輯連結開啟一般編輯介面，變更只存在接收者的目前工作狀態，可手動儲存、匯出 JSON 或重新產生分享連結。
6. 唯讀分享桌機版（大於 `820px`）在左上顯示完整清單。手機版（`820px` 含以下）預設只顯示底部「路線清單・N 站」，`N` 是目的地數。點標題切換 `body.share-list-open`。抽屜內容上限 `38vh`，內部清單上限 `27vh`，展開時隱藏距離／時間摘要，收合後恢復。點站點後手機抽屜收合，地圖 `flyTo()` 該目的地（縮放至少 15），約 500ms 後打開標記資訊。桌機點站同樣定位並開啟標記。

分享是包含名稱及座標的可解碼快照，唯讀是前端介面模式。網址片段通常不隨 HTTP 頁面請求傳送，但收件者可讀取資料，瀏覽器會把路線座標送給 OSRM 計算。發送網址或 QR Code 前請確認其中沒有要保密的行程。

## 匯入、匯出與備份

| 格式 | 現行行為 |
| --- | --- |
| CSV | 可匯入與匯出。解析逗號分隔的 UTF-8，支援 BOM、雙引號、引號內逗號及雙引號跳脫。匯出為 UTF-8 BOM、CRLF，所有儲存格以雙引號包住 |
| JSON | 可匯入與匯出，也是本機與正式頁之間搬移路線的備份格式。`data()` 輸出 `app: 'RoutePlannerNoGoogle'`、`version: '1.5.5'`、`routeName`、`createdAt`、`updatedAt`、`destinations`、`viasByLeg` 及 `route: {distance, duration}`。`restore()` 支援舊 `name`／`savedAt`，依點位重新規劃 |
| GPX | 匯出 GPX 1.1。目的地與定位點為 `wpt`，路線為 `trk`／`trkseg`／`trkpt` |
| KML | 匯出 KML 2.2。目的地與定位點為 `Placemark`／`Point`，路線為 `Placemark`／`LineString` |
| PDF | 「列印 / PDF」叫用瀏覽器列印，列印樣式保留地圖與摘要，隱藏工具列、側欄與唯讀抽屜 |

`line()` 優先使用 OSRM 的路線幾何。若尚未算出或服務失敗，GPX／KML 會使用點位依序直線相連。GPX／KML 在本版只有匯出入口。`dl()` 使用 Blob 與瀏覽器下載，XML 名稱使用 `xml()` 跳脫。

CSV 匯入需要緯度與經度欄。緯度接受 `緯度`、`lat`、`latitude`，經度接受 `經度`、`lng`、`lon`、`longitude`。選填名稱接受 `名稱`、`地點名稱`、`name`，類型接受 `類型`、`type`，排序接受 `編號/路段`、`編號`、`路段`、`order`、`leg`。欄名去掉空白並轉小寫後比對。目的地依編號排序，缺少編號時依目的地出現順序。類型包含「定位」或等於 `via`／`waypoint` 時，依 `路段-順序` 歸入已排序目的地間的路段。匯入會取代目前點位，有既有目的地時先詢問。路線名稱與建立時間沿用目前工作狀態。

```csv
類型,編號/路段,名稱,緯度,經度
目的地,1,台北車站,25.0478,121.5170
定位點,1-1,"路線定位點, 經過中山",25.0524,121.5207
目的地,2,行天宮,25.0631,121.5337
```

## 外部服務與限制

| 服務 | 端點與目前行為 | 維護時的限制 |
| --- | --- | --- |
| OpenStreetMap 圖磚 | `https://tile.openstreetmap.org/{z}/{x}/{y}.png`，Leaflet 保留地圖署名，第一次載入錯誤會顯示狀態 | 維持可見署名、有效 Referer 和正常快取。禁止預抓或大量下載圖磚，服務無 SLA。[圖磚使用政策](https://operations.osmfoundation.org/policies/tiles/) |
| Nominatim 公開搜尋 | `https://nominatim.openstreetmap.org/search`，每次最多 5 筆、偏好繁中，依小寫查詢快取 7 天。每頁以約 1050ms 間隔做節流 | 公開服務每秒最多 1 次，以全站應用流量計。單頁節流與每台瀏覽器的快取無法保證全站合規。維持使用者主動查詢、有效識別、署名與可切換服務的能力。禁止 autocomplete、系統化查詢及提交私人資料。[公開服務政策](https://operations.osmfoundation.org/policies/nominatim/) |
| OSRM 示範服務 | `https://router.project-osrm.org/route/v1/driving/`。加點、刪除、換序、拖曳等操作及分享初始化都可觸發規劃 | 示範端點限合理的非商業用途，每秒最多 1 次，無可用性、延遲或資料更新保證。50 點是本工具上限，端點另可回 `TooBig` 等錯誤。大量測試使用模擬回應。[示範端點政策](https://github.com/Project-OSRM/osrm-backend/wiki/Demo-server)、[OSRM API](https://project-osrm.org/docs/v5.24.0/api/) |
| jsDelivr 與瀏覽器能力 | CDN 提供 Leaflet CSS／JS 與 qrcodejs。定位、剪貼簿、壓縮及下載依瀏覽器能力 | Leaflet 載入失敗時顯示錯誤。QR 程式載入失敗時仍可複製連結或提供 JSON。定位與 Clipboard API 使用 HTTPS／localhost 並依授權運作。使用真機驗證掃碼及手機剪貼簿 |

## 本機開發

在 Windows PowerShell 執行下列指令，然後以 Chrome 或 Edge 開啟 `http://127.0.0.1:8000/`。按 `Ctrl+C` 停止伺服器。一般互動預覽需要能連線至上述外部服務。若使用 `py` 啟動器，可將 `python` 換成 `py -3`。

```powershell
Set-Location -LiteralPath 'O:\CHATGPT\route_plan'
python -m http.server 8000 --bind 127.0.0.1
```

編輯 `index.html` 後重新整理。檢查開發人員工具的 Console、Network 與裝置模式。使用範例點位，並用新的瀏覽器設定檔或無痕視窗分別開啟兩種分享網址。若有 Node.js，可在根目錄執行下列選用語法檢查，將內嵌腳本透過標準輸入送給 Node。

```powershell
$html = Get-Content -LiteralPath '.\index.html' -Raw -Encoding utf8
$script = [regex]::Match($html, '(?s)<script>\s*(.*?)</script>').Groups[1].Value
if (-not $script) { throw '找不到內嵌 JavaScript' }
$script | node --check
```

## 測試清單

本次交接已通過 Node.js 語法檢查與本機 Chrome 驗收。測試涵蓋唯讀／可編輯分享、舊連結、QR Code 產生、820／821px 斷點及手機抽屜的選站動作，也驗證 JSON 匯入、CSV 往返、GPX／KML 匯出及 50／51 點的匯入上限。Chrome 測試使用模擬圖磚和 OSRM 回應。真機掃碼、定位權限、剪貼簿及實際 API 的整合測試仍可依下表驗收。

| 情境 | 應驗證的結果 |
| --- | --- |
| 載入與定位 | 地圖、圖磚署名及標記正常。座標搜尋直接產生結果。另做一次地名查詢，檢查搜尋結果、快取及錯誤提示。瀏覽器定位測允許和拒絕 |
| 編輯與路段 | 至少兩個目的地才有規劃，橘色定位點不占目的地編號。拖曳、改名、刪除中間點、換序時的取消與確認、定位點排序與清除皆符合預期 |
| 上限與異常資料 | 手動／CSV／JSON／分享測試 50、51 點，用模擬 OSRM。加點在 51 點被拒。檢查無效座標、空座標、錯誤路段、空檔、破損 JSON／分享 token，以及失敗前後的路線狀態 |
| 分享與相容性 | 測試唯讀、可編輯、缺少 `e`、`g.`、`r.` 和舊無前綴 token。確認可編輯端改名後原連結快照不變。確認 QR、複製、超長連結提示，並以真機掃碼 |
| 唯讀版面 | 桌機 `821px` 顯示左上清單。手機 `820px` 及 `390px` 預設收合，展開可捲動且摘要隱藏。點選目的地後收合、摘要恢復、地圖定位並開啟標記。另測長中文名稱與 19 站以上的清單 |
| 格式與備份 | CSV 測 UTF-8 BOM、CRLF、含逗號／雙引號的名稱與往返順序。JSON 測路線名、舊欄位與定位點往返。以 GIS 或 XML 工具檢查 GPX／KML 的座標順序、特殊字元及直線備援。檢查列印與 JSON 跨來源搬移 |
| 第三方異常 | 模擬 OSRM `NoRoute`、HTTP 錯誤及回應順序顛倒，檢查摘要和舊路線。模擬地圖圖磚、QR CDN 和 Nominatim 不可用 |

## 日後部署

1. 於 `O:\CHATGPT\route_plan\` 確認 `git status` 與 `git remote -v`，唯一 `origin` 為 `https://github.com/squall021/route-planner.git`。先取得遠端 `main` 並比對目前修改，依 [測試清單](#測試清單) 完成本機驗收。
2. 網站入口為根目錄 `index.html`。需要改版時同步標題、上限提示及 JSON 版本。依使用者授權提交及推送至 `origin/main`，保留遠端既有提交。
3. 在獨立儲存庫 [Settings → Pages](https://github.com/squall021/route-planner/settings/pages) 選擇分支發布，來源為 `main`、目錄為 `/ (root)`。[GitHub Pages 發布來源設定](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)
4. 待 Pages 建置與部署完成，核對部署提交、正式 HTML 版本及分享抽屜。網站網址為 `https://squall021.github.io/route-planner/`，與原工具相同，既有 `#share=` 片段可沿用。
5. 原 `squall021.github.io` 的 `route-planner/` 資料夾及工具箱首頁繼續保留在原儲存庫。需要回復發布時，在獨立儲存庫還原對應修改並循相同流程部署。

## 風險與待辦

1. **OSRM 重複請求與競態。** `restore()` 會呼叫 `route()`，`initSharedRoute()` 又呼叫一次。快速編輯也可重疊送出請求，現行沒有取消或防止較舊結果覆蓋。優先合併重複請求，加入節流與失效回應保護；若仍依賴示範端點，先評估整體用量與服務政策。
2. **匯入應先完整驗證。** CSV 的路段存在性是在 `S.d`／`S.v` 被替換後才驗證，錯誤可能留下部分修改。空 CSV 座標目前可能被 `Number('')` 當成 0。JSON 會過濾無效目的地，但 `viasByLeg` 仍依原索引復原，可能造成路段對不上。改善時先建暫存資料，確認座標、路段與容量後再一次寫入。
3. **公開服務依賴。** 地名搜尋節流只覆蓋單頁，OSRM 沒有全站節流。若流量增加或端點政策變更，評估可設定的端點、自建服務或具有配額與 SLA 的供應商。保留 OSM 署名並檢查手機抽屜會不會遮住它。
4. **分享與瀏覽器相容性。** 可補上 `aria-expanded`、`aria-controls`、收合後焦點與鍵盤驗收。可處理壓縮資料損壞、過大輸入及 QR 產生失敗；分享名稱應持續在 HTML／XML 邊界跳脫。剪貼簿備援目前未檢查 `execCommand('copy')` 回傳值，失敗時可能仍顯示已複製。
5. **檔案輸出細節。** 清除全部會清空圖層與顯示，但 `S.distance`／`S.duration` 沒同步歸零，JSON 可能帶出舊摘要。CSV 若匯入試算表，應評估以 `=`、`+`、`-` 或 `@` 開頭的名稱被解譯成公式的情況，修正時兼顧 CSV 往返相容性。

上述待辦供後續開發使用，本次以保存上游原始碼作為交接基準。
