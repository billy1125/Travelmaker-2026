---
name: update-itinerary
description: 討論完行程異動後，歸檔舊版並同步更新 itinerary.md 與 days/ 每日檔案
disable-model-invocation: true
allowed-tools: Read Write Edit Bash PowerShell
---

每次討論有行程變動後，執行此技能，將最新內容同步到文件。

## 格式規範

更新前請先閱讀（路徑相對於此 SKILL.md 所在目錄）：
- `reference/itinerary_format.md` — itinerary.md 的格式規範
- `reference/day_format.md` — days/ 每日檔的格式規範

## 執行步驟

### 1. 歸檔變更前的版本

在修改任何檔案之前，先把「本次即將異動」的檔案複製一份到 `archive/`：

1. 用 Bash 取得時間戳：`date +%Y%m%d-%H%M`
2. 建立 `archive/{時間戳}/` 目錄（若同一分鐘內已存在，沿用該目錄）
3. 將即將異動的檔案依原有相對路徑複製過去，例如：
   - `itinerary.md` → `archive/20260802-1030/itinerary.md`
   - `days/0830.md` → `archive/20260802-1030/days/0830.md`

規則：

- **只複製本次會改到的檔案**，不整包複製專案
- 新增的日期檔（原本不存在）不需歸檔
- 歸檔是複製不是搬移，原檔留在原位繼續編輯

### 2. 更新 itinerary.md

依 `reference/itinerary_format.md` 將確認的行程變更寫入，範圍包括：

- 標頭（天數、日期、航班、住宿安排）
- 行程總覽表格（日期用 `MM/DD` 補零、住宿、摘要）
- 表格日期欄連結格式：`[MM/DD](days/MMDD.md)`
- 行程原則（如有調整）

### 3. 更新每日檔案

依 `reference/day_format.md` 更新或新增 `days/` 中受影響的檔案：

- 檔名格式：`MMDD.md`（月日補零，不含標題）
- 若已存在：直接更新內容
- 若為新日期：建立新檔案
- 若某日移除：不刪除舊檔，在檔案開頭加註 `> 此日已從行程移除（YYYY/MM/DD）`

### 4. 回報修改清單

完成後條列：
- 歸檔目錄路徑，與其中包含的檔案
- 各檔異動狀態（新增 / 修改）

## 限制

- 不修改 `reference/itinerary_format.md` 與 `reference/day_format.md`（相對於此技能目錄）
- 不刪除或覆寫 `archive/` 內既有的歸檔目錄
- 所有內容使用繁體中文
