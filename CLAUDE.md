# CLAUDE.md

此檔案提供給 Claude Code（claude.ai/code）在此專案目錄中工作時的指引。

## 這個專案是什麼

這是一個個人旅遊規劃目錄，記錄 2026 年日本岡山・倉敷・尾道之旅，並非軟體專案。目錄結構：

- `itinerary.md` — 主行程總覽（含各日連結），包括行程的使用者基本設定，關於行程的基本資料都從這裡開始
- `days/` — 每日詳細行程，各自獨立的 Markdown 檔案（命名格式：`MMDD.md`）
- `assets/` — 參考資料（電子機票、飯店確認書、網友資源索引）
- `transportations/` — 各路段交通指南（命名格式：`起訖點代碼_路段.md`）
- `README.md` — 簡易專案說明

## 工作流程

1. 執行 `/discuss-itinerary`
   - 自動讀取全部行程、交通指南與參考資源，確保討論有完整的資料基礎
   - 若使用者提供外部網址，抓取後納入；需即時資訊則網路搜尋

2. 與使用者討論
   - 依據前述資料與使用者的限制，完成希望的行程內容
   - 如果使用者特定指出以代理人進行討論，必須參考 **代理人** `.claude/agent/` 章節說明

3. 執行 `/update-itinerary`
   - 同步更新 `itinerary.md` 與 `days/` 內的受影響檔案

4. 整理資料（強制：凡本次討論中有抓取任何網站內容，即須執行）
   - 執行 `/save-website-abstract`
   - 逐一確認每個已抓取的網站：若 `assets/website_abstract.md` 尚無該網站的摘要，補寫並歸入對應分類
   - 依 `assets/reference_website.md` 的分類（交通、美食、購物、景點、行李準備）寫入 `assets/website_abstract.md`

## 資料來源

- `assets/電子機票.pdf` — 去回程電子機票（台灣虎航 JEU19B）
- `assets/Super Hotel Okayama Station Higashiguchi.pdf` — 岡山住宿訂房確認書（08/28–08/31，3 晚）
- `assets/Yutori by b hotel - 1Br Apartment for 3Ppl in a quiet.pdf` — 尾道住宿訂房確認書（08/31–09/05，5 晚）
- `assets/reference_website.md` — 網頁參考資源索引（交通、美食、購物、景點、行李準備連結），*由使用者維護*
- `assets/website_abstract.md` — Claude 整理的網站摘要（依 `reference_website.md` 分類歸入）
- `transportations/OKJ_OkayamaStation.md` — 岡山桃太郎機場 ↔ 岡山站利木津巴士指南

## 技能

- `.claude/skills/discuss-itinerary/` — 行程討論前準備：讀取全部行程、交通指南與參考資源後再討論（每次討論前執行）
- `.claude/skills/update-itinerary/` — 行程更新技能與格式規範（`reference/` 資料夾在此）
- `.claude/skills/save-website-abstract/` — 將討論中抓取的新網站整理成摘要，寫入 `assets/website_abstract.md`（討論後、有新網站時執行）
- `.claude/skills/tw-opendata-transportation/` — 查詢台灣交通批次資料集（台鐵、高鐵、捷運、公路客運等路線、票價、時刻表）

## 代理人

- `.claude/agent/trip-planner.md` — 旅遊行程規劃師：先讀取現有 `itinerary.md` 與 `days/`，再依使用者需求產出行程草稿（預設存成 `itinerary_draft.md`）
- `.claude/agent/trip-reviewer.md` — 行程審查員：讀取 `itinerary_draft.md`（或 `days/*.md`）後逐項審查可行性，依嚴重／警告／建議三級回報，只審查不修改

## 編輯行程時的原則

- 行程的限制與條件，設定在`itinerary.md`
- 去回程日若有對應交通指南，在標頭加入連結（參考 `days/0828.md` 格式）
- 新增或修改行程時，執行 `/update-itinerary` 同步 `itinerary.md` 與 `days/` 對應檔案
- 討論中只要有抓取網站內容，結束前必須執行 `/save-website-abstract`；凡摘要尚無記錄的網站，均需補寫後歸入 `assets/website_abstract.md`
