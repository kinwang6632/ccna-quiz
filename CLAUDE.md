# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案概觀

這是一個單一 HTML 檔案構成的 CCNA 認證考古題測驗網頁應用（收錄第 1–1300 題），沒有建構流程，也沒有 npm/套件依賴（僅透過 CDN 載入 Google Fonts）。全部 HTML、CSS、JavaScript 與題庫資料都內嵌在 [ccna-quiz.html](ccna-quiz.html) 這一個檔案裡（約 13.9MB，多數體積來自題目內嵌的 base64 圖片）。

## 常用指令

- 本地預覽：直接用瀏覽器開啟 `ccna-quiz.html`，或起一個靜態伺服器
  ```bash
  python3 -m http.server 8000
  ```
- 沒有 build / lint / test 指令（純靜態單檔專案，無 package.json 或任何建構工具）

## 佈署（GitHub Pages）

- 網址：https://kinwang6632.github.io/ccna-quiz/ （儲存庫 `kinwang6632/ccna-quiz`，公開）
- 來源：`main` 分支根目錄（`/`），沒有 GitHub Actions workflow；推送到 `main` 後 GitHub 會自動重新發佈，約 1 分鐘生效。
- 根目錄的 `index.html` 只是用 meta refresh 導向 `ccna-quiz.html`，主程式仍是 `ccna-quiz.html`；若更改主檔名要同步修改導向。
- `.nojekyll` 讓 Pages 直接提供原始檔、不經 Jekyll 處理，不要刪除。
- 查看發佈狀態：
  ```bash
  gh api repos/kinwang6632/ccna-quiz/pages/builds/latest --jq .status
  ```
- 儲存庫是公開的：PDF 原檔與 `*.bak-*` 備份檔已由 `.gitignore` 排除，新增檔案前確認不要把題庫來源或個人資料推上去。
- 在 Pages 上執行時沒有 `window.claude`，作答紀錄只存在瀏覽器 `localStorage`，不會跨裝置同步（需要時用錯題本匯出／匯入）。

## 架構

`ccna-quiz.html` 內部大致分三段：

1. **`<style>`（檔案開頭）**：以 CSS 自訂屬性（`--bg`、`--ink`、`--accent` 等，定義在 `:root`）做主題化，深色主題透過 `<html data-theme="...">` 切換（見 `applyTheme()`）。
2. **`const DATA = [...]`（第 378 行）**：全部 1300 題的題庫，整包資料在同一實體行，是檔案體積的主要來源。每題物件形如 `{n, type, opts|dd, ans, stem, img?, code?, ex?, k?}`，且這一行內容是合法 JSON，可直接用 `json.loads` 解析：
   - `type: "mc"` 為選擇題，`k` 為應選數量（`k > 1` 即複選題，`ans` 為多個 key 相連，如 `"AD"`）；`opts` 為 `[key, text]` 選項陣列，選項文字可含 `\n` 表示多行設定指令。
   - `type: "dd"` 為拖放題：`kind: "match"` 為一對一配對，`dd.targets` 為 `{label, ans}`（`ans` 為對應的 item 文字）；`kind: "group"` 為分類型，`dd.groups` 為 `{name, ans: [...]}`。允許部分 item 不屬於任何目標（「Not all options are used」的干擾項）。
   - `img` 欄位是內嵌 base64 WebP 圖片，只放拓樸圖、表格、GUI 截圖等真正的圖形附圖，不要把題幹／選項文字裁進去。
   - `code` 欄位是等寬字型顯示的 CLI 輸出／設定檔／JSON 等純文字附圖；能轉成文字的附圖優先放 `code`，比圖片清楚且檔案小很多。
   - `ex` 欄位是解析，格式為 `[[0|1, 文字], ...]`（`0` 為段落、`1` 為縮排條列）。來源題庫標註「題庫給的答案是 X」「猜」「背」或有爭議的題目，原註記要保留在 `ex` 裡，並說明採用哪個答案與理由。
   - 若要新增／修改題目，建議寫 Python 或 Node 腳本解析並改寫這個 JS 陣列，不要直接用一般文字編輯器整行手動改，容易破壞語法。改完要驗證 `n` 從 1 起連續無缺號，並同步更新標題、選單小標、`.mnote` 與首頁 `home-sum` 中寫死的總題數與拖放題數。
   - **沒有標準答案的題目（實作題）**：若某題屬於需要實際在裝置或模擬器（如 Packet Tracer）上操作驗證的實作題，資料裡不要硬塞假答案——`mc` 題給 `opts: []` 且不要給 `ans`，`dd` 題不要讓 `dd.targets[].ans` / `dd.groups[].ans` 補滿。前端 `hasAnswer(q)`（約第 590 行）會自動偵測「答案不完整」的題目，並在畫面上把它顯示成「實作題」提示卡（只顯示題目本身，不提供選項作答與批改），不會計入作答進度／錯題本/統計。
   - **實作題不列入測驗、但保留在總覽**：啟動時依 `hasAnswer` 算出 `LABS`（實作題題號）與 `QUIZ_ALL`（可測驗題號）。`poolFor()` 只從 `QUIZ_ALL` 取題，`adopt()` 讀入舊存檔的 `order` 時也會濾掉實作題，所以測驗清單裡永遠不會出現實作題；測驗成績的分母也用 `QUIZ_ALL.length`。總覽的題號格仍顯示全部 1300 題，實作題加上 `.port.lab`（灰字）標示，點選後以 `adhoc` 方式直接查看、不影響測驗進度。功能選單有「實作題列表」區塊（`#menuLabs`，於 `renderStats()` 內產生），點選項目會關閉選單並以 `setMode('quiz', n)` 直接查看該題。新增或移除實作題時不需改這些邏輯，只要資料符合「答案不完整」的規則即可自動歸類。
3. **應用邏輯（第 375 行起）**：原生 JavaScript，無框架、無外部 JS 套件。
   - 主題專區：`TOPICS`（緊接在 `RANGE_MIN/MAX` 之後）以題號清單定義主題（目前只有 `wireless` 無線網路，170 題），`S.topic` 記錄目前啟用的主題（`null` 為全部），`poolFor()` 透過 `inTopic()` 篩選出題；總覽右欄的「無線網路專區」區塊（`topicSectionHTML()`，含進度摘要、進入／離開專區按鈕與題目清單）與選單設定的開關都呼叫 `setTopic()`，專區模式下總覽的非專區題號加上 `.port.off` 淡化。新增題目若屬於無線網路，要把題號加進 `TOPICS.wireless.ids`。
   - 全域狀態物件 `S`：`{mode, pos, order, shuffle, range, count, topic, quiz, wrong, mastered, star, theme, wrongAt, keyAt, updatedAt}`，對應目前模式、作答進度、答題紀錄、錯題、已精熟題號、重點背誦題號（`star`：`{題號: 加入時間}`）等。
   - 錯題本／重點背誦位置：`S.wrongAt`、`S.keyAt` 記錄兩個模式最後停留的題號（`render()` 時更新並存檔），切回該模式或重新載入時 `buildWrongList()` / `buildKeyList()` 透過 `posAt()` 回到該題（已移出則停在下一題）；只有「重新整理錯題本」「清空錯題本」「清空重點背誦」才傳 `fresh = true` 從頭開始並清空本輪紀錄。測驗模式直接沿用存檔的 `S.pos.quiz`，總覽的「繼續測驗」「前往專區測驗」按鈕不帶 `data-n`，回到上次停留的題號而不是第一個未作答題；只有變更出題範圍／隨機／主題（`rebuildOrder(true)`）或重設測驗才回到第一題。總覽頁的捲動位置存在 `homeScroll`（離開總覽、`pagehide`、切到背景時寫入 localStorage key `ccna-home-scroll`，不同步雲端），回到總覽或重新載入時 `renderHome(homeScroll)` 捲回原處。另有「回到置頂」`goFirst()`：測驗／錯題本／重點背誦的底部列 `#firstBtn`（⇤ 第一題，手機上也顯示文字）跳回清單第一題，總覽捲離頁首後右下角出現 `#toTop`（↑ 回到頂端）；`Home` 鍵在各模式都呼叫 `goFirst()`。
   - 四種模式對應頂部分頁：`home`（總覽，含題號總覽格 + 錯題摘要 + 重點背誦清單 + 統計）、`quiz`（測驗）、`wrong`（錯題本）、`key`（重點背誦）。
   - 重點背誦：每題卡片右上角的 ☆ 按鈕（或快捷鍵 `S`）呼叫 `toggleStar()` 加入／移除；`key` 模式清單由 `buildKeyList()` 依題號建立，可正常作答（答錯同樣記入錯題本），也可按「直接看答案」（`peek()`）不批改直接顯示答案與解析，本輪紀錄存在 `keyDone`（不存檔）。
   - 儲存與同步：`save()` 一律先寫入 `localStorage`（key `ccna120-v3`，並相容讀取舊版 `ccna120-v1`/`v2`）；若當前是在 Claude Artifact 環境執行（偵測到 `window.claude.use('db'/'user')`），會另外 debounce（1200ms）同步到雲端文件 `data/users/{uid}/state`（`connectCloud()` / `flush()`），讓紀錄能跨裝置沿用；否則僅存在瀏覽器本機。
   - 錯題本匯入／匯出：功能選單「紀錄管理」中的按鈕，`exportWrong()` 把 `S.wrong` 與 `S.star` 下載成 JSON，`importWrongText()` 讀回並與現有錯題合併（次數加總、保留較新的作答紀錄，略過題庫中不存在的題號）。
   - 渲染方式是手動重繪：修改 `S` 後呼叫 `render()`（或針對性呼叫 `renderHome()` / `renderPanel()` / `renderStats()`）整段重新產生 innerHTML，沒有虛擬 DOM 或框架層。
   - 拖放題（dd）邏輯集中在 `slotsOf` / `slotCorrect` / `correctPlacement` / `placeItem` / `renderBoard`，同時支援滑鼠拖曳與觸控「點選再點空格」兩種互動方式。
   - 鍵盤快捷鍵綁在檔案尾端的 `document.addEventListener('keydown', ...)`：`M` 開關選單、`←`/`→` 換題、`A`–`F` 選答、`Enter` 送出或顯示答案、`S` 加入／移除重點背誦。`Home` 回到第一題（總覽則捲回頁首）。

## 注意事項

- 此檔案設計成雙模式運作：可獨立在一般瀏覽器開啟使用，也可作為 Claude Artifact 執行並啟用雲端同步（同步邏輯會先偵測 `window.claude` 是否存在，再決定是否啟用）。修改同步相關程式碼時要留意兩種環境都要能正常降級運作。
- 第 374 行的題庫資料是單一超長實體行，用一般文字編輯器開啟或搜尋取代時要特別小心；改題庫內容時優先用 Edit 工具做局部片段替換，而非整檔重寫。
