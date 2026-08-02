# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 這個專案是什麼

個人旅遊規劃資料庫，記錄 2026/08/28–09/05 日本岡山・倉敷・尾道 9 天 8 晚之旅。內容全部是 Markdown 與 PDF，**不是軟體專案**：沒有建置、測試、lint 指令，唯一的「執行」動作是編輯 Markdown 並用 git 版控。

## 資料架構

三層結構，改動時必須同層與跨層一起維持一致：

1. **`itinerary.md`（單一事實來源）** — 行程總覽索引，含航班、住宿分段、行程總覽表格與**行程原則**。所有硬性限制（早出發時間、住宿不換點、班機緩衝）都寫在「行程原則」，任何行程調整必須先對照這一節，不符合就是要先跟使用者確認的改動。此檔只放一行摘要，細節一律放 `days/`。
2. **`days/MMDD.md`（每日細節）** — 0828–0905 共 9 檔。`itinerary.md` 表格的日期欄以 `[MM/DD](days/MMDD.md)` 連回。
3. **`transportations/` 與 `assets/`（支援資料）** — 交通指南與參考資料，被上面兩層引用。

其他檔案：

- `luggage_items.md` — 行李清單（checkbox 格式，含鋰電池新規等航空限制）
- `itinerary_draft.md` — `trip-planner` 代理人的輸出路徑，**平時不存在**，只在呼叫該代理人時產生；屬草稿而非正式行程，正式內容一律以 `itinerary.md` 與 `days/` 為準，草稿內容併入後即可刪除
- `archive/{YYYYMMDD-HHMM}/` — `/update-itinerary` 在每次改動前自動留下的變更前副本，維持原有相對路徑；**唯讀，不刪除也不覆寫既有目錄**

## 每日行程的確認狀態

`days/MMDD.md` 開頭若有這一列，代表該日資料已逐項查證過：

```markdown
> ✅ **YYYY.MM.DD 資訊已初步確認**
```

- **沒有這一列的日子，資料尚未查證**，提出建議前要重新確認營業時間、票價與班次
- 完成一日查證後，主動詢問使用者是否加上橫幅；日期寫查證當天
- **commit message 要記錄該日已確認**：標題與第一段寫明是哪一天、何時確認，其餘異動列在後面
- `.claude/settings.json` 設有 PreToolUse hook，執行 `git commit` 前會列出 `days/` 內尚未標記 ✅ 的日期，提醒審核

## 檔案的擁有者

這點決定哪些檔案可以改：

| 檔案 | 擁有者 | 規則 |
|------|--------|------|
| `assets/reference_website.md` | 使用者 | **只讀不寫**，網址索引由使用者維護 |
| `assets/website_abstract.md` | Claude | 由 `/save-website-abstract` 寫入，依交通／美食／購物／景點／行李準備分類 |
| `.claude/skills/*/reference/*.md` | 格式規範 | **不修改**，是被引用的規範來源 |
| `itinerary.md`、`days/`、`transportations/` | 共同 | 經討論確認後由 `/update-itinerary` 更新 |

## 工作流程

1. `/discuss-itinerary` — 討論前強制先讀完 `itinerary.md`、`transportations/` 全部檔案、`days/` 全部檔案、`assets/reference_website.md` 與 `assets/website_abstract.md`。**未讀完不得提出任何行程建議**。
2. 與使用者討論；需要即時資訊（票價、時刻表、公休日）用 WebSearch 查證，使用者給網址則 WebFetch。
3. `/update-itinerary` — 有行程變動時，先把即將異動的檔案歸檔到 `archive/{YYYYMMDD-HHMM}/`，再同步 `itinerary.md` 與受影響的 `days/MMDD.md`；若異動涉及新的交通節點，會接著呼叫 `build-transportation`。
4. `/build-transportation` — 盤點各日行經的車站、港口、巴士站、機場與市內交通系統，**先列出待建置節點清單並取得使用者同意**，再為同意的節點建立 `transportations/` 檔案並在對應 `days/` 標頭加上連結。也可單獨呼叫。
5. `/save-website-abstract` — **只要本次討論抓取過任何網站就必須執行**，把尚無摘要的網站補寫進 `assets/website_abstract.md`。

## 格式規範重點

完整規範在 `.claude/skills/update-itinerary/reference/`（`itinerary_format.md`、`day_format.md`、`transportation_format.md`）與 `.claude/skills/save-website-abstract/reference/website_abstract_format.md`，編輯前先讀。跨檔案共通的重點：

- 檔名月日補零：`days/0903.md`、`transportations/OKJ_OkayamaStation.md`（`OKJ` 為岡山桃太郎機場 IATA 代碼，格式為 `起訖點代碼_路段.md`）
- 交通指南**不放時刻表**（只寫班距與首末班提醒，時刻表給官方連結）；車站構造圖 PDF 放 `assets/` 並以 `../assets/xxx.pdf` 連結；**同一條動線寫在同一個檔案**，不因換交通工具而拆檔
- 景點一律加 Google Maps 搜尋連結：`https://www.google.com/maps/search/?api=1&query={搜尋詞}`（搜尋詞用日文原名）
- 每日檔只有 `## 行程` 必填；`## 時間表` 在抵達日與回程日必填；`## 交通` 僅跨城市或多段轉乘日使用
- **備案**（雨天、體力不足、班機延誤）寫在 `## 行程` 區塊內用粗體標示，不另開章節
- 檔頭 `> ⚠️` 放需事先預約的項目；檔尾 `> ⚠️` 放出發前需再確認的營業時間與公休日
- 某日從行程移除時**不刪檔**，在檔案開頭加註 `> 此日已從行程移除（YYYY/MM/DD）`
- 網站摘要每條寫具體事實（金額、時間、班次、公休日），不寫廣告語；抓取失敗（403／逾時）就跳過該 URL 並在回報中說明，不建立空條目

## 代理人

`.claude/agents/trip-planner.md`（規劃草稿，輸出 `itinerary_draft.md`）、`.claude/agents/trip-reviewer.md`（只審查不修改，依嚴重／警告／建議三級回報）。使用者明確要求以代理人進行時才使用。

## 環境限制

`.claude/settings.json` 禁止直接執行 `python`／`pip`，僅允許 `conda run -n claudecode:*`。需要跑 Python 時走 conda 環境。
