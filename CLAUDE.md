# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案概觀

這是一個單一 HTML 檔案構成的 CCNA 認證考古題測驗網頁應用（收錄第 1–1100 題），沒有建構流程，也沒有 npm/套件依賴（僅透過 CDN 載入 Google Fonts）。全部 HTML、CSS、JavaScript 與題庫資料都內嵌在 [ccna-quiz.html](ccna-quiz.html) 這一個檔案裡（約 12MB，多數體積來自題目內嵌的 base64 圖片）。

## 常用指令

- 本地預覽：直接用瀏覽器開啟 `ccna-quiz.html`，或起一個靜態伺服器
  ```bash
  python3 -m http.server 8000
  ```
- 沒有 build / lint / test 指令（純靜態單檔專案，無 package.json 或任何建構工具）

## 架構

`ccna-quiz.html` 內部大致分三段：

1. **`<style>`（檔案開頭）**：以 CSS 自訂屬性（`--bg`、`--ink`、`--accent` 等，定義在 `:root`）做主題化，深色主題透過 `<html data-theme="...">` 切換（見 `applyTheme()`）。
2. **`const DATA = [...]`（第 359 行）**：全部 1100 題的題庫，整包資料在同一實體行，是檔案體積的主要來源。每題物件形如 `{n, type, opts|dd, ans, stem, img?, code?, ex?, k?}`，且這一行內容是合法 JSON，可直接用 `json.loads` 解析：
   - `type: "mc"` 為選擇題，`k` 為應選數量（`k > 1` 即複選題，`ans` 為多個 key 相連，如 `"AD"`）；`opts` 為 `[key, text]` 選項陣列，選項文字可含 `\n` 表示多行設定指令。
   - `type: "dd"` 為拖放題：`kind: "match"` 為一對一配對，`dd.targets` 為 `{label, ans}`（`ans` 為對應的 item 文字）；`kind: "group"` 為分類型，`dd.groups` 為 `{name, ans: [...]}`。允許部分 item 不屬於任何目標（「Not all options are used」的干擾項）。
   - `img` 欄位是內嵌 base64 WebP 圖片，只放拓樸圖、表格、GUI 截圖等真正的圖形附圖，不要把題幹／選項文字裁進去。
   - `code` 欄位是等寬字型顯示的 CLI 輸出／設定檔／JSON 等純文字附圖；能轉成文字的附圖優先放 `code`，比圖片清楚且檔案小很多。
   - `ex` 欄位是解析，格式為 `[[0|1, 文字], ...]`（`0` 為段落、`1` 為縮排條列）。來源題庫標註「題庫給的答案是 X」「猜」「背」或有爭議的題目，原註記要保留在 `ex` 裡，並說明採用哪個答案與理由。
   - 若要新增／修改題目，建議寫 Python 或 Node 腳本解析並改寫這個 JS 陣列，不要直接用一般文字編輯器整行手動改，容易破壞語法。改完要驗證 `n` 從 1 起連續無缺號，並同步更新標題、選單小標、`.mnote` 與首頁 `home-sum` 中寫死的總題數與拖放題數。
   - **沒有標準答案的題目（實作題）**：若某題屬於需要實際在裝置或模擬器（如 Packet Tracer）上操作驗證的實作題，資料裡不要硬塞假答案——`mc` 題給 `opts: []` 且不要給 `ans`，`dd` 題不要讓 `dd.targets[].ans` / `dd.groups[].ans` 補滿。前端 `hasAnswer(q)`（約第 543 行）會自動偵測「答案不完整」的題目，並在測驗畫面把它顯示成「實作題」提示卡（只顯示題目本身，不提供選項作答與批改，直接按「下一題」略過即可），不會計入作答進度／錯題本/統計。
3. **應用邏輯（第 360 行起）**：原生 JavaScript，無框架、無外部 JS 套件。
   - 全域狀態物件 `S`：`{mode, pos, order, shuffle, range, count, quiz, wrong, mastered, theme, updatedAt}`，對應目前模式、作答進度、答題紀錄、錯題、已精熟題號等。
   - 三種模式對應頂部分頁：`home`（總覽，含題號總覽格 + 錯題摘要 + 統計）、`quiz`（測驗）、`wrong`（錯題本）。
   - 儲存與同步：`save()` 一律先寫入 `localStorage`（key `ccna120-v3`，並相容讀取舊版 `ccna120-v1`/`v2`）；若當前是在 Claude Artifact 環境執行（偵測到 `window.claude.use('db'/'user')`），會另外 debounce（1200ms）同步到雲端文件 `data/users/{uid}/state`（`connectCloud()` / `flush()`），讓紀錄能跨裝置沿用；否則僅存在瀏覽器本機。
   - 錯題本匯入／匯出：功能選單「紀錄管理」中的按鈕，`exportWrong()` 把 `S.wrong` 下載成 JSON，`importWrongText()` 讀回並與現有錯題合併（次數加總、保留較新的作答紀錄，略過題庫中不存在的題號）。
   - 渲染方式是手動重繪：修改 `S` 後呼叫 `render()`（或針對性呼叫 `renderHome()` / `renderPanel()` / `renderStats()`）整段重新產生 innerHTML，沒有虛擬 DOM 或框架層。
   - 拖放題（dd）邏輯集中在 `slotsOf` / `slotCorrect` / `correctPlacement` / `placeItem` / `renderBoard`，同時支援滑鼠拖曳與觸控「點選再點空格」兩種互動方式。
   - 鍵盤快捷鍵綁在檔案尾端的 `document.addEventListener('keydown', ...)`：`M` 開關選單、`←`/`→` 換題、`A`–`F` 選答、`Enter` 送出或顯示答案。

## 注意事項

- 此檔案設計成雙模式運作：可獨立在一般瀏覽器開啟使用，也可作為 Claude Artifact 執行並啟用雲端同步（同步邏輯會先偵測 `window.claude` 是否存在，再決定是否啟用）。修改同步相關程式碼時要留意兩種環境都要能正常降級運作。
- 第 359 行的題庫資料是單一超長實體行，用一般文字編輯器開啟或搜尋取代時要特別小心；改題庫內容時優先用 Edit 工具做局部片段替換，而非整檔重寫。
