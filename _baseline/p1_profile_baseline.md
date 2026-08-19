---
type: test_baseline
scope: persona profile 讀寫接縫（Phase 1 read-through lazy migration 之前）
captured_at: 2026-08-19T05:32Z
captured_by: kiara (wake#15)
plan: ucl_core:Docs~/zh-Hant/Plan/Plan_Persona_Registry_Retirement.md §4 Phase 1 / §8.2 / §8.4 / §8.6 / §8.7
env: LY 專案 / AgentCommands branch=LY / Phase 0 已全落地（接縫兩端＋A+B 快照＋寫入審計）
rulings: 酒館 seq 12447（kiara 四題）→ seq 12448（summit 四題全答）
---

# P1 行為快照基線 — Template 殼的 profile 欄位讀寫（Phase 1 施工前）

> 用途：Phase 1 把 identity 欄搬進 `letters/<persona>/profile/` 之後，拿本檔逐項 diff。
> **「行為沒變」必須拿本檔比對，不是宣稱。**
> 儀式那一腿（登入→晚安）的基線在同目錄 `p0_morning_baseline.md`，本檔不重跑（理由見最後一節）。

## 0. 起點乾淨度（Phase 1 的前提）

| 量 | 實測 | 意義 |
|---|---|---|
| 全庫 `letters/*/profile/` 目錄 | **0 個** | 沒有任何人已被部分遷移；乾淨起點 |
| code 裡的 `profile/` 讀寫實作 | **0 處** | 唯一命中是 `UCL_DiscordInboundDaemon` 的 Discord API 欄名，無關 |

## 1. python 端三段 fallback（`_lib/persona_profile.py`）

### tier-1 live（Editor 開著，主路徑）

```
source_info() → {'source': 'live', 'snapshot_at': ''}
pool_names() → 21 人，含 Template
get_raw("Template") → 無 _source / 無 _snapshot_at / 無 _field_sources
```

`get_routing("Template")` → `{"agent": "Template", "model": "claude-opus-5", "actual_agent": "ClaudeCode"}`

`get_identity("Template")` → 8 欄全數命中：
`layer_role` / `forked_from`(null) / `fork_lineage`([]) / `forked_at`(null) /
`created_at`("2026-08-12T12:50:54.838Z") / `identity_vector`(64 維) / `vector_history`(6 筆) / `email`("")

### tier-2 snapshot（`UCL_PP_SKIP_CMD=1`）

```
source_info() → {'source': 'snapshot', 'snapshot_at': '<快照 generated_at>'}
get_raw / get_routing 回傳值均帶 _source="snapshot" ＋ _snapshot_at
```

✅ **兩態不得同形（§8.7 Tim 五輪拍板）成立** —— live 無標記、snapshot 有標記，實測分得開。
**Phase 1 不得破壞這條。**

### tier-3 local-parse

未在本次觸發（快照存在）。summit seq 12448 第五格拍板：
**tier-3 在 Phase 1 之後會讀不到 `profile/` 的新值，刻意不修**
（給 python 長 profile/ 解析器＝第二解析器還魂）；`_source="local-parse"` 已宣告「可能舊」。
⇒ 驗收時**不要**把「tier-3 讀到舊值」當回歸。

## 2. 快照形狀（`AwakenInit/_persona_profile_snapshot.json`）

| 量 | Phase 1 前 |
|---|---|
| top-level keys | `generated_at` / `identity_fields` / `personas` / `pool` / `routing_fields` |
| `routing_fields` | `agent, model, actual_agent`（3） |
| `identity_fields` | `layer_role, forked_from, fork_lineage, forked_at, created_at, identity_vector, vector_history, email`（8） |
| `pool` 長度 | 21 |
| `personas.Template` 欄數 | 15 |
| per-persona 來源標記 | **無** |

⇒ Phase 1 要加的 `_field_sources`（欄→`profile`\|`legacy`，summit 採納本小姐的提案）
**在本檔是「不存在」** —— 它出現＝Phase 1 生效的可觀測證據，也是 §8.4「log 歸零」判準的量尺。

## 3. 寫入接縫（`Cmd PersonaProfile op=set`）—— 落點與連動

實測：`op=set persona=Template field=email value=p1-baseline@test.invalid actor=kiara:p1-baseline`

| 觀測點 | Phase 1 前的結果 |
|---|---|
| 值落在哪 | **`AwakenInit/personas/Template.json`（舊源）** |
| `letters/Template/profile/` | **未生成** |
| 審計 `_persona_write_audit.jsonl` | append 一行（ts/persona/fields/actor/reason）✓ |
| 快照 | 同秒重寫，`generated_at` 推進、新值可讀 ✓ |
| 回傳 | `old_value` / `new_value` 兩欄 |

⇒ Phase 1 後這格要翻成：值落 `profile/email.md`、**舊源不得被回寫**（§8.4 鐵則）。

### fail-loud 複驗（Phase 1 不得弄壞的保證）

省略 `actor` 重跑同一筆：

```
真正的 exit = 2
訊息：[PersonaProfile] set 失敗：actor 與 reason 必填（§8.6）—— 寫入要能回答「是誰、憑什麼」；匿名寫入不收
```

- 值**未落地**（仍是前一筆的值）✓
- 審計檔**未增行**（被擋的寫入不留審計）✓

> 🩸 量這一格時本小姐自己踩了一次 summit 記過的坑：`cmd | tail; echo $?` 抓到的是 `tail` 的退出碼
> （量到 exit=0，差一步就記成「fail-loud 沒作用」）。改成不接管線重量才拿到 exit=2。
> **量退出碼不要接管線** —— 這條在本案的驗收裡會一直遇到。

### 量完已復原

`email` 已 set 回原值空字串（`old_value=p1-baseline@test.invalid` → `new_value=`）。

## 4. ⚠ Phase 1 設計必須先處理的一格：「缺欄」與「欄在但空」同形

`op=set` **不能刪欄**，所以 `absent → present-empty` 是單向的；
而 C# `GetIdentity` 用 `jd.Contains(f)` 判存在 ⇒ **兩者在讀取層本來就不同形**
（缺欄不出現在 identity dict，空欄會出現）。

這件事撞上 §8.4 的 lazy migration 觸發條件（「`profile/` 缺欄 ⇒ 當場遷」）：

- legacy **根本沒有那一欄** ⇒ 遷移時該生一個空的 `profile/<field>.md`，還是**什麼都不生**？
- 若選「生空檔」，一次無關的寫入（把 absent 變成 present-empty）就能讓一個**從來不存在的欄**
  在 `profile/` 長出一個看起來是資料的空檔。
- 若選「什麼都不生」，則「已遷完」與「這人沒這欄」在 `profile/` 目錄上同形 ——
  §8.4 的「log 歸零＝都遷完了」就量不準。

⇒ 本小姐的傾向：**只遷「legacy 有這個 key」的欄**，並讓 `_field_sources` 明確記三態
（`profile` / `legacy` / `absent`）—— 讓「沒有」自己有名字，而不是靠檔案不存在來暗示。
待與 @summit 確認（不擋工，但實作前要有答案）。

⚠ 附帶事實：本次量測時 `personas/Template.json` 相對 HEAD 已有一筆
**先於本次量測存在**的 `"email":""` 差異（Phase 0 email 歸位那輪留下的），不是本次寫入造成 ——
本次寫入淨值為零。記在這裡免得後人把它算到 Phase 1 頭上。

## 5. 為什麼本檔不重跑登入→晚安

Phase 1 只搬 **identity 8 欄**。morning patch-write 動的是
`wake_count / status / availability / last_active / actual_agent / model`
—— 全是活體欄與 routing 欄，**沒有一欄在 Phase 1 的搬遷範圍內**，
且儀式讀取一律經 `GetRaw`（合併層的落點）⇒ 儀式那一腿的對照組用 `p0_morning_baseline.md` 即可，
再跑一次只會多寫一次 lock 與一則酒館廣播。

⚠ 但 **Phase 1 完工後必須照 p0 全流程實跑一次 Template**（summit 交接的鐵律二），
確認合併層沒有從 `GetRaw` 底下改變儀式行為。**本節是「現在不跑」的理由，不是「不必跑」。**
