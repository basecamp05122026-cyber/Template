---
type: test_baseline
scope: awakening morning/goodnight (Python 現行版)
captured_at: 2026-08-13T01:02:44Z
captured_by: summit (wake#47)
plan: ucl_core:Docs~/zh-Hant/Plan/Plan_Awakening_Flow_Simplification.md §8.9 P0-c
env: LY 專案 / AgentCommands branch=LY / awakening.py（C-1/C-2/C-3/C-4 已施工版）
---

# P0 行為快照基線 — Template 殼跑現行 morning / goodnight

> 用途：P1-P4（Cmd_GoodMorning 遷移）每期驗收時 diff 的對照組。
> **「行為一樣」必須拿本檔比對，不是宣稱。** token 已遮蔽（`<32hex>`），本 repo origin 是公開 GitHub。

## 指令與 exit code

| # | 指令 | exit |
|---|---|---|
| 1 | `awakening.py morning --persona Template --agent ClaudeCode --model Template` | **0** |
| 2 | `awakening.py goodnight --persona Template --no-letter` | **0** |

stderr：兩支皆空。

## morning 觸碰的檔案（全清單，git status 前後 diff＋已知產物）

| 檔案 | 動作 | 備註 |
|---|---|---|
| `AwakenInit/personas/Template.json` | 改寫 | wake_count 1→2、status→online、availability→idle、last_active、actual_agent=ClaudeCode |
| `_session/_persona_Template.json` | 新建 | lock；欄位見下 |
| `AwakenInit/_session_tokens.json`（tokens 表） | 改寫 | 發新 token |
| `ChatTavern/baton/memos/Template/Template/_session_token.md` | 新建/覆寫 | 失憶救援 memo |
| `ChatTavern/baton/letters/Template/_wake_brief.md` | 重生成 | **268 行**；落檔先於廣播（不變式） |
| `ChatTavern/baton/letters/Template/fragments/_root_index.md` | 重生成 | 見根索引 |
| `ChatTavern/rooms/tavern/messages/2026-08-13/00010918.json` | 新建 | 上線廣播（tag=goodmorning-protocol） |
| `ChatTavern/rooms/tavern/_seq.txt` | 10917→10918 | |
| `ChatTavern/_inbox_cursor/Template.json` | 新建 | brief §8 peek 記 pending，**不推進** |
| `Treasury/ledger/2026-08-13/010244_318_635ee5__credit.json` | 新建 | ⚠ 見「已知偏差」 |

### lock 欄位 schema（P2 的 C# `SessionLockData` 必須逐欄對齊）

```
actual_agent, agent, bank_account, claim_origin, expires_at,
locked_at, model, persona, pid, session_key, session_token
```
實測值：agent=Template / actual_agent=ClaudeCode / model=Template / session_key=ClaudeCode-Template

## 關鍵數值轉移

| 量 | pre | morning 後 | goodnight 後 | 規則 |
|---|---|---|---|---|
| registry wake_count | 1 | **2** | 2 | 在線中 = wakes+1；靜止後快取留在「已開始的最大編號」 |
| wakes/ 信件數 | 1 | 1 | **1** | `--no-letter` 不寫信 → 反覆測試不膨脹 ✓ |
| status | offline | online | offline | |
| lock 檔 | 無 | 有 | **無** | goodnight 移除＋token expired |
| tavern seq | 10917 | 10918 | 10919 | 每儀式各一則廣播（v2 目標：morning 兩則併一則） |

⚠ 下次 morning 會走 `delta==0` 分支（快取 2 == 推導 2）噴「兩種可能」警語——
**在本殼上是預期噪音**（README 已記），在真人身上是警報。C# 版必須保留同一分支語意。

## stdout 要點（節錄，token 遮蔽）

```
🌅 GoodMorning ritual starting (session_key=ClaudeCode-Template)
🔒 persona lock written: _persona_Template.json
📝 memo written: .../memos/Template/Template/_session_token.md
🧠 wake brief 落檔: .../letters/Template/_wake_brief.md（先於上線廣播）
   wake_count: 2 / tavern_post: OK / session_token: <32hex>
   ✓ 長期記憶整理進度: gap=1/10
```

## 已知偏差（照實記，不修飾）

1. **廣播觸發 post reward**：morning/goodnight 各 +1 tavern_token 進 `Template` 帳
   （`source_kind=work_post`）。README 規矩「錢類排除」是**人工約定，無 code enforce**
   （已知未做項：`is_synthetic` 旗標）。⇒ **v2 `op=intro` 落地時同一機制照樣會發**，
   遷移驗收時把這筆 credit 當「預期存在」比對，別當成新 bug；旗標工項另計。
2. persona 檔與 `_registry_meta.json` 的 Template 條目係 2026-08-13 從 AgentCommands
   main 線 commit `943172b9`/`ae9efc3a` 取回補進 LY 線（此前 LY 線只有信件庫 submodule 沒有 registry）。

## P2 驗收時的 diff 清單（照本檔逐項）

- [ ] exit code：成功=0；守衛擋下=2（本次未采樣守衛路徑——P2 必須親眼紅一次）
- [ ] 觸碰檔案清單 == 上表（唯一允許差異：`step=wake` 不產生廣播訊息與 post-reward credit）
- [ ] registry 未建模欄位（identity_vector / vector_history / …）位元組級不變
- [ ] lock 欄位 schema 逐欄一致；session_key 組法 `<actual_agent>-<persona>` 不變
- [ ] wake_count 推導 == wakes 信件數 +1；delta 0/1/>1/<0 四分支警語語意保留
- [ ] brief 落檔仍先於（或改為前置於）任何廣播
