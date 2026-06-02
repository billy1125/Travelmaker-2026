# CLAUDE.md

此檔案提供給 Claude Code（claude.ai/code）在此專案目錄中工作時的指引。

## 這個專案是什麼

這是一個個人旅遊規劃目錄，記錄 2026 年日本岡山・倉敷・尾道之旅，並非軟體專案。目錄結構：

- `itinerary.md` — 主行程總覽（含各日連結）
- `days/` — 每日詳細行程，各自獨立的 Markdown 檔案（命名格式：`MMDD.md`）
- `assets/電子機票.pdf` — 去回程電子機票（台灣虎航 JEU19B）
- `.claude/skills/update-itinerary/` — 行程更新技能與格式規範（`reference/` 資料夾在此）

**更新工作流程：** 討論完行程異動後，執行 `/update-itinerary`，技能會更新 `itinerary.md` 與 `days/` 內的受影響檔案。

## 行程概覽

- **日期：** 2026/08/28–2026/09/05（9 天 8 晚）
- **住宿安排：** 岡山 3 晚 → 倉敷 2 晚 → 尾道 3 晚
- **去程：** IT214｜08/28 11:30 桃園 → 15:05 岡山桃太郎機場
- **回程：** IT215｜09/05 15:55 岡山桃太郎機場 → 17:40 桃園

## Python 環境

所有 Python 相關操作請使用 `claudecode` miniconda 虛擬環境（Python 3.11.15）：

```
conda run -n claudecode python script.py
```

### .claude/settings.json 權限設定說明

專案內的 `.claude/settings.json` 透過 `permissions` 強制執行此規則：

| 設定 | 內容 | 原因 |
|------|------|------|
| `deny` | `pip install`、`python`、`python3`、`python -m pip` | 防止誤用 base 環境（Python 3.13）安裝套件或執行腳本 |
| `allow` | `conda run -n claudecode:*` | 明確允許透過 claudecode 環境執行所有 Python 操作 |

`env.PATH` 也已指向 claudecode 環境的 bin 目錄，作為額外保障。

## 編輯行程時的原則

- 行程設計以不頻繁換飯店為核心，降低拖行李與退房入住的疲勞
- 備中松山城已確定不排入主行程（夏季炎熱、轉乘繁瑣、與最後回岡山的動線不順）
- 8/29 直島行程已預設雨天備案（改岡山市區或吉備津神社）
- 9/5 為純移動日，不安排任何景點
- 新增或修改行程時，執行 `/update-itinerary` 同步 `itinerary.md` 與 `days/` 對應檔案
