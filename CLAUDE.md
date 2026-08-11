# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 這個專案是什麼

個人旅遊規劃資料庫，記錄 2026/08/28–09/05 日本岡山・倉敷・尾道 9 天 8 晚之旅。內容全部是 Markdown 與 PDF，**不是軟體專案**：沒有建置、測試、lint 指令，唯一的「執行」動作是編輯 Markdown 並用 git 版控。

## 檔案與資料夾架構

三層結構，改動時必須同層與跨層一起維持一致：

- **`itinerary.md`（行程總覽索引）**：含基本航班、住宿分段、行程總覽表格與**行程原則**。所有硬性限制（早出發時間、住宿不換點、班機緩衝）都寫在「行程原則」，任何行程調整必須先對照這一節，不符合就是要先跟使用者確認的改動。此檔只放一行摘要，細節一律放 `days/`。
- **`days/`（每日細節）**：記錄每日行程要點，包括交通、時間表、行程、費用預估與注意事項。
- **`transportations/` (交通節點)**：記錄行程中重要的交通節點或節點間資訊，例如主要出入境與轉機機場資訊、轉乘與路經車站等資訊。共有三種內容：
  - **重要交通起終點連通資訊**：重要的行程中交通說明，例如：`OKJ_OkayamaStation.md` 代表由出入境機場岡山機場至岡山車站的說明，由於這影響主要行程，因此特別獨立檔案說明。
  - **單一車站**：提供包括站點的建築結構與內部環境、與行程有關之基本時間提示等。
  - **市內交通系統**：與單一車站類似，但主要是說明該市內交通的使用方式，例如當地地鐵、公車系統等。

- **`hotel/`（旅宿指南）**：一間旅宿一個檔案，記錄訂房事實（確認碼、PIN、日期、房型、人數、房價、取消政策）、館內設施與時間、房內設備、特殊規定、周邊生活機能、本次行程使用規劃，以及**四個情境的英日文聯絡信件**（行前確認／入住與鑰匙／抵達時間變更／行李寄放與其他需求）。訂房事實一律以 `assets/` 的訂房確認單 PDF 為準，網路查到的資料不覆蓋它。
- **`photospots/`（攝影景點）**：**一個城市一個檔案**，檔名為 PascalCase 英文城市名（例：`Onomichi.md`）。記錄拍攝點的位置、Google Maps 連結、**周邊地標**、構圖與時段建議、拍攝注意事項，並附「與本次行程的對應」一節，標明哪些點落在既有動線上、哪些需要額外加排。**這一層只記錄拍攝資訊，不改動行程**；要排進行程仍須經討論後由 `/update-itinerary` 寫入 `days/`。

> `days/` 由 `itinerary.md` 的行程總覽表格連結；`transportations/`、`hotel/` 與 `photospots/` 不掛在 `itinerary.md` 下，而是由各 `days/yyyyMMDD.md` 標頭連結過去（`photospots/` 亦可僅由 `README.md` 索引）。

其他檔案：

- `README.md` — 對外索引，包括 `itinerary.md`、`luggage_items.md`，含每日行程、交通指南、旅宿指南與攝影景點的完整連結表；新增或移除 `days/`、`transportations/`、`hotel/`、`photospots/` 檔案時要同步更新。
- `assets/`（支援資料） — 主要是使用者提供的額外資料，例如電子機票、旅館預定紀錄等，被上面各層所引用；另含使用者維護的網址索引 `reference_website.md`、Claude 寫入的網站摘要 `website_abstract.md`，以及 `build-transportation` 下載的車站構造圖 PDF，擁有者見下方表格。

  > 本環境沒有 `pdftoppm`，Read 工具**無法直接讀 PDF**。需要讀訂房確認單等 PDF 時，用 conda 環境的 PyMuPDF（`fitz`）寫一支腳本抽文字到暫存 `.txt` 再讀。`conda run` 不支援含換行的 `-c` 參數，腳本要寫成檔案；也不要讓腳本 print 中日文，`conda run` 的 stdout 是 cp950 會爆。
- `luggage_items.md` — 行李清單（checkbox 格式，含鋰電池新規等航空限制），原則上僅有清單與簡單提示，不放與行程、景點有關資訊。
- `archive/{YYYYMMDD-HHMM}/` — `/update-itinerary` 在每次改動前自動留下的變更前副本，維持原有相對路徑；**唯讀，不刪除也不覆寫既有目錄**

## 工作流程

各技能的執行細節寫在自己的 `SKILL.md`，這裡只定順序與時機：

1. `/discuss-itinerary` — 每次討論行程前先跑，讀完全部資料才能提建議。
2. 與使用者討論；需要即時資訊（票價、時刻表、公休日）用 WebSearch 查證，使用者給網址則 WebFetch。
3. `/update-itinerary` — 有行程變動時跑，會自行歸檔並視需要接著呼叫 `build-transportation`。
4. `/build-transportation` — 有新交通節點時跑，也可單獨呼叫。
5. `/build-hotel` — 有新旅宿或訂房內容變動時跑，也可單獨呼叫。
6. `/save-website-abstract` — **本次討論只要抓取過任何網站就必須跑**。

## 每日行程的確認狀態

`days/yyyyMMDD.md` 開頭若有這一列，代表該日資料使用者已經「**初步**」逐項查證過：

```markdown
> ✅ **YYYY.MM.DD 資訊已初步確認**
```

若有這一列，代表該日資料使用者已經「**完整**」逐項查證過，可以視為不會再修改的檔案內容：

```markdown
> ✅ **YYYY.MM.DD 資訊已確認**
```

- **沒有這一列的日子，資料尚未查證**，提出建議前要重新確認營業時間、票價與班次
- 橫幅**由使用者決定**，Claude 不代填、不代改；完成一日查證後主動詢問是否加上，日期寫查證當天
- 隨時可用這兩行分別列出兩種狀態（與 commit hook 同一組判斷）：
  ```bash
  grep -LE "^> ✅" days/*.md              # 完全未查證
  grep -lE "^> ✅.*初步確認" days/*.md     # 只到初步，仍需複查
  ```
- **commit message 要記錄該日已確認**：標題與第一段寫明是哪一天、何時確認，其餘異動列在後面
- `.claude/settings.json` 設有 PreToolUse hook，執行 `git commit` 前會分別列出「尚未查證」與「僅初步確認」的日期，提醒審核

## 檔案的擁有者

這點決定哪些檔案可以改：

| 檔案 | 擁有者 | 規則 |
|------|--------|------|
| `assets/reference_website.md` | 使用者 | **只讀不寫**，網址索引由使用者維護 |
| `assets/website_abstract.md` | Claude | 由 `/save-website-abstract` 寫入，依交通／美食／購物／景點／行李準備分類 |
| `.claude/skills/*/reference/*.md` | 格式規範 | 平時視為規範來源**只讀不寫**，**使用者明確要求時才改** |
| `itinerary.md`、`days/`、`transportations/` | 共同 | 經討論確認後由 `/update-itinerary` 更新 |
| `hotel/` | 共同 | 經討論確認後由 `/build-hotel` 更新；訂房事實以 `assets/` 的訂房 PDF 為準 |
| `photospots/` | 共同 | 查證後直接更新，**不需經 `/update-itinerary`**；但把拍攝點排進行程仍要走 `/update-itinerary` |

## 技能清單

`.claude/skills/` 目前有六個技能：

| 技能 | 用途 | 呼叫方式 |
|------|------|----------|
| `discuss-itinerary` | 討論前讀完全部行程、交通指南與參考資源 | 使用者 `/discuss-itinerary`，Claude 也可自行判斷使用 |
| `update-itinerary` | 歸檔舊版後，同步 `itinerary.md` 與 `days/yyyyMMDD.md` | **只能由使用者呼叫**（frontmatter 設 `disable-model-invocation: true`） |
| `build-transportation` | 盤點交通節點，建立或補充 `transportations/` 指南 | 使用者呼叫，或由 `update-itinerary` 接續呼叫 |
| `build-hotel` | 盤點旅宿，建立或補充 `hotel/` 指南與英日文聯絡信件 | 使用者呼叫，Claude 也可自行判斷使用；建檔前一定要先取得使用者同意 |
| `save-website-abstract` | 把本次抓取的網站整理成摘要寫入 `assets/website_abstract.md` | 使用者呼叫；抓過網站就該執行 |
| `tw-opendata-transportation` | 台灣交通部 OpenData 批次資料（台鐵、高鐵、捷運、公路客運、民航的路線／站點／票價／時刻表／運量統計） | **暫時不使用**，查的是台灣端的交通資料，本次行程日本段用不到，**目前不呼叫**；需要查桃園機場聯外或台鐵、高鐵時再由使用者明確指示啟用 |

## 格式規範重點

基本格式規範細節定義於技能 `update-itinerary` 與 `save-website-abstract` 之中，可藉由修改兩項技能來調整所需行程格式。`hotel/` 的格式規範同樣放在 `update-itinerary/reference/`，由 `build-hotel` 以相對路徑引用。以下僅是主要的重要規範重點內容摘錄：

- `days/` 檔名格式為 `yyyyMMDD.md`，例如 `days/20260903.md`
- `transportations/` 檔名分路段、單一車站、市內交通系統三種，命名規則見 `transportation_format.md`
- `hotel/` 檔名為 PascalCase 英文旅館名，連鎖品牌加地點後綴，例如 `hotel/SuperHotelOkayamaHigashiguchi.md`，規則見 `hotel_format.md`

## 代理人

> ⚠️ **不能直接引用**。僅有使用者明確要求以代理人進行時才使用。代理人設計尚未完善，可能會有意外之錯誤。

- `.claude/agents/trip-planner.md`（規劃草稿，輸出 `itinerary_draft.md`）。
- `.claude/agents/trip-reviewer.md`（只審查不修改，依嚴重／警告／建議三級回報）。

## 環境限制

`.claude/settings.json` 禁止直接執行 `python`／`pip`，僅允許 `conda run -n claudecode:*`。需要跑 Python 時走 conda 環境。
