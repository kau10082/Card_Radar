# CardRadar 💳

[![Claude Skill](https://img.shields.io/badge/Claude-Skill-D97757?logo=anthropic&logoColor=white)](https://claude.com/claude-code)
[![Notion](https://img.shields.io/badge/Notion-database-000000?logo=notion&logoColor=white)](https://www.notion.so/)
[![MCP](https://img.shields.io/badge/MCP-Notion-3b82f6)](https://modelcontextprotocol.io/)
[![Credit-Card Rewards](https://img.shields.io/badge/credit--card-rewards-22c55e)](https://github.com/topics/credit-card-rewards)
[![Docs](https://img.shields.io/badge/docs-繁中%20%7C%20EN-lightgrey)](#cardradar-)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> 你的個人信用卡回饋哨兵——丟一個消費情境，立刻告訴你「該刷哪張卡、實際拿多少」。
>
> Your personal credit-card rewards sentinel — describe a purchase, and it instantly tells you **which card to use and how much you'll actually earn**.

CardRadar 是一個 [Claude](https://claude.com/claude-code) Skill。它把你手上每一張卡、每一個通路的回饋規則，集中記在你自己的 Notion 資料庫裡；查詢時即時讀取、依「實拿金額」排名，給你一個可以直接行動的答案。Skill 本身**不儲存任何回饋數字、不寫死任何帳號 ID 或個人資料**——所有機敏與個人化內容都留在本機、被 `.gitignore` 排除的檔案中。

CardRadar is a [Claude](https://claude.com/claude-code) Skill. It keeps every card-and-channel rewards rule in *your own* Notion database, reads it live on each query, ranks options by **actual cash earned** (not headline rate), and hands you an answer you can act on immediately. The Skill itself **stores no reward figures and hard-codes no account IDs or personal data** — everything sensitive lives in local, git-ignored files.

---

## 為什麼是「實拿」 · Why "actual earnings"

回饋率會騙人，因為幾乎每張卡都有每月上限。CardRadar 一律用下面這條公式排名，而不是比誰的百分比漂亮：

Reward *rates* are misleading because almost every card caps the monthly payout. CardRadar always ranks by the formula below, never by whose percentage looks prettiest:

```
實拿 actual = min(消費金額 amount × 回饋率 rate, 每月上限 monthly cap)
```

刷得越少，越接近純比率比較；刷得越多，上限越關鍵。
The less you spend, the closer it is to a pure rate comparison; the more you spend, the more the cap decides the winner.

## 三條鐵則 · Three core rules

1. **算實拿，不比率** — 以 `min(金額 × 率, 上限)` 排名。
   **Actual over rate** — rank by `min(amount × rate, cap)`.
2. **只認你在用的支付** — 不在你支付清單裡的行動支付，其綁定加碼一律不計入。
   **Only your wallets count** — bonuses tied to mobile wallets you don't use are excluded.
3. **誠實標狀態** — 新增／更新時有疑惑當場向你釐清，確認後直接寫「生效」；已失效不列入、即將到期會提醒。
   **Honest status** — anything unclear is clarified with you on the spot before writing, then saved as active; expired rows are dropped, soon-to-expire ones flagged.

## 運作方式 · How it works

```
/card 觸發 trigger
   ↓ 讀 Notion 規則表（逐列 fetch）read the Notion rules DB (row by row)
   ↓ 套用通路特例與支付限制 apply channel quirks & wallet limits
   ↓ 依「實拿」排名 rank by actual earnings
   ↓ 回報最優卡 + 封頂點 + 注意事項 report best card, cap point, caveats
```

## 安裝與設定 · Setup

1. 把 `cardradar/` 資料夾放進你的 Claude skills 目錄（例如 `~/.claude/skills/cardradar/`）。
   Drop the `cardradar/` folder into your Claude skills directory (e.g. `~/.claude/skills/cardradar/`).

2. 複製範本為本機設定，並填入你自己的值：
   Copy the templates to local config and fill in your own values:
   ```bash
   cp cardradar/config.example.json  cardradar/config.local.json
   cp cardradar/profile.example.json cardradar/profile.local.json
   ```
   - `config.local.json` — 你的 Notion 規則表 `data source id`、DB 連結，以及選填的逐列 page ID 索引。
     Your Notion `data source id`, DB link, and an optional row-level page-ID index.
   - `profile.local.json` — 你的持卡資料：實際卡片、使用的行動支付與配對、會員等級／戶況、點數形式。
     Your card data: actual cards, wallets in use and their pairings, membership tier/status, point types.

3. 在 Notion 建好「信用卡回饋規則表」資料庫——欄位定義見 [`cardradar/SKILL.md`](cardradar/SKILL.md) 的 *DB schema* 一節。
   Create the "credit-card rewards" database in Notion — see the *DB schema* section in [`cardradar/SKILL.md`](cardradar/SKILL.md) for the exact fields.

> ⚠️ `config.local.json` 與 `profile.local.json` 已被 `.gitignore` 排除，**永遠不會**上傳。請勿把真實 ID 或個資寫進 `*.example.json` 或 `SKILL.md`。
>
> ⚠️ `config.local.json` and `profile.local.json` are git-ignored and **never** committed. Never put real IDs or personal data into `*.example.json` or `SKILL.md`.

## 用法 · Usage

| 指令 Command | 作用 What it does |
|---|---|
| `/card <消費情境 scenario>` | 查最優卡 Find the best card — 例 e.g. `/card 全聯怎麼付`、`/card 高鐵票6000刷哪張` |
| `/card 新增 <卡名 card>` | 查官方資料、新增回饋列（有疑惑當場釐清後寫入）Look up official terms, add reward rows (clarifying anything unclear first) |
| `/card 更新 <卡名/通路>` | 重查並更新既有列（有疑惑當場釐清）Re-check and update existing rows (clarifying anything unclear) |
| `/card 失效 <列>` | 活動結束標已失效 Mark ended promotions as expired |

## 檔案結構 · Layout

```
.
├─ .gitignore
├─ README.md
└─ cardradar/
   ├─ SKILL.md               # 通用邏輯（公開）· generic logic (public)
   ├─ config.example.json    # Notion ID 範本 · ID template
   ├─ profile.example.json   # 個人化資料範本 · profile template
   ├─ config.local.json      # 你的真實 ID（git-ignored）· your real IDs
   └─ profile.local.json     # 你的真實持卡資料（git-ignored）· your real card data
```

## 隱私 · Privacy

這個 repo 是公開的，但**不含任何個人資料**：所有帳號 ID、卡片清單、會員等級與支付偏好都只存在本機的 `*.local.json`，並由 `.gitignore` 擋下。公開的只有通用邏輯與空白範本——任何人都能複製、填入自己的資料來用。

This repo is public but **contains no personal data**: all account IDs, card lists, membership tiers, and wallet preferences live only in local `*.local.json` files, kept out by `.gitignore`. What's public is the generic logic and blank templates — anyone can clone it, fill in their own data, and use it.
