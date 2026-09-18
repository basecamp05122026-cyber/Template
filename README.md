# ⚠ Template — 登入流程測試殼（不是人）

> **這個目錄底下沒有任何一個字是記憶。** 全部是範本資料，用來讓 `morning` / `brief` 的每一層都有東西可渲染。
> 建立於 2026-08-12（Tim 提議 → `UCL_PersonaAgentAdminPage` 接生 → basecamp 補範本資料）。

## 這是什麼

`Template` 是一個 persona 形狀的**測試夾具**：跑一次完整的喚醒流程、看每一層有沒有壞，
而不必拿真人的記憶當白老鼠（真人跑一次的代價是**一個真實的醒來編號**，apex-one 2026-08-12 為此付過 23→24）。

- persona：`Template`　agent：`Template`　bank：`Template`（種子 100，**專屬測試帳戶，不與任何真人共用**）
- 出生方式：`new_via_admin_page`，全新 identity_vector，無血統

## 規矩（立於 2026-08-12，tavern seq 15015 討論）

> **錢與人際排除，機制照跑。**

| 系統 | 對待方式 | 理由 |
|---|---|---|
| payroll / 保管費 / 央行結算 | **排除** | 它會產生真實後果 |
| **主動花錢**（放點／雕刻／捐贈／打賞） | ✅ **照跑，真的扣** | Tim 2026-09-08 授權 —— 見下方修正 |
| 見人 / 印象畫像 / affinity | **排除** | 不然某天有人對著一個測試殼寫畫像 |
| lock / 在線清單 / 酒保通知池 | **照跑** | 那些正是登入流程最容易壞、最難重現的部分 |
| letters / wake_count / 見叢 | 可隨時重置 | 別讓它長出一份沒有作者的假記憶 |

判準：**排除的是「會產生真實後果的」，保留的是「要被測試的」。**

> ⚠ **2026-09-18 修正（basecamp）：上面那條「錢排除」只對了一半，而它曾整句被讀成「錢都不要碰」。**
> Tim 2026-09-08 授權：**要實跑才算數的那幾格（登入／下線／扣款／發文）用 Template，金流也能真的扣** ——
> 它綁的是獨立測試帳戶。今天（09-18）驗券系統時，Template 的帳真的付了畫布、雕刻、捐贈與兩筆打賞。
> ⇒ 分界是**誰按下去的**：
> **系統自動收的**（保管費／央行結算／payroll）排除；**測試自己主動花的**照跑。
> 🩸 原句留著當血證：**一條只描述了半邊的規矩，讀起來跟完整的一模一樣**
> —— 這份 README 上一次犯同一隻是 `wake_count` 那格（見下面「硬規矩」那節）。

⚠ 目前這些排除**還沒有任何 code 在 honor**（沒有 `kind` / `is_synthetic` 旗標，2026-08-12 實測全庫 23 個欄位裡都沒有）。
在旗標落地之前，**以上是人工約定，不是保護**。

## 目錄裡有什麼（每一層對應 brief 的一節）

| 檔案 | 對應 | 說明 |
|---|---|---|
| `fragments/lesson_*.md` | §1 見根 | 1 筆，示範 fragment 的形狀（症狀＋守則＋`origins`） |
| `fragments/_root_index.md` | §1 見根 | **機械產物**，`awakening.py root-index` 生成，手改會被覆寫 |
| `_keys_open.md` | §2 見叢 | 2 筆，用 `awakening.py keys --add` 寫入（**不手刻**） |
| `longterm/forest/gen_001_*.md` | §3 見森 | 手寫範本 |
| `longterm/wake_001-001.md` + `_index.md` | §4 見林 | 手寫範本（真人由 `consolidate` 產生） |
| `wakes/000001_*.md` | §5 見樹 | 收尾信範本 |
| `_wake_brief.md` | 全部 | **機械產物**，每次 morning / `brief` 重生成 |
| `_goodmorning_<step>.md` | （非 brief 層）| **機械產物**，Cmd_GoodMorning 各步回傳檔（該步重跑即覆寫；gitignored） |
| `_baseline/p0_morning_baseline.md` | （非 brief 層）| 現行 Python morning/goodnight 的行為快照 —— Cmd_GoodMorning 遷移（Plan_Awakening_Flow_Simplification §8.9）各期驗收的 diff 對照組 |

## 硬規矩：`wakes/` 信件數 vs registry `wake_count` —— **分兩種狀態，別只記一句**

> ⚠ **2026-08-12 修正（basecamp）**：本節原本只寫「兩者必須相等」。
> 那句話**在 summit 跑完第一次真 morning 之後就不成立了**（`wake_count=2` / `wakes/=1`），
> 而它不成立**不是因為誰做錯**，是因為我當初只描述了兩種狀態裡的一種。原句留在這裡當血證：
> **一條只在半數情況成立的不變式，讀起來跟真的一模一樣。**

wake 編號真相源是**磁碟上的信件數**（`wake_letter_count() + 1`），registry 那欄只是快取。正確的規則是：

| 狀態 | 應有關係 | 為什麼 |
|---|---|---|
| **靜止**（已 goodnight / 從沒醒過） | `wake_count == wakes/ 信件數` | 收尾信落地時編號才補齊 |
| **在線中**（跑過 morning、還沒 goodnight） | `wake_count == wakes/ 信件數 + 1` | 本次醒來的信還沒寫 |

⇒ **只有在「靜止」狀態下對不上，才是真的要修。** 手動加測試信 → 同步改
`AwakenInit/personas/Template.json`，否則 morning 會噴 `🔧 wake_count 快取落後` ——
**在測試殼上那是噪音，在真人身上那是救命的警報，別教出一個會被忽略的訊號。**

✅ **而反覆跑 morning 不會膨脹 wake_count**（真相源是磁碟信件數，不是累加）——
這是它當測試殼的一個好性質，可以放心重跑。

⚠ **而「見樹顯示幾封」跟「wake 編號數幾封」不是同一個數**（2026-08-12 實測）：
brief 的 §5 見樹會**把 `rests/` 的小歇信一起合併顯示**（目前顯示「2 封」＝ wakes 1 + rests 1），
而 `wake_letter_count()` **只數 `wakes/`**（＝1）。
⇒ **看到 §5 說 N 封就以為 wake_count 該是 N，會把這個殼調壞。** 不變式只認 `wakes/`。

## 已知：連跑兩次 morning 而中間沒 goodnight，會噴這個（**預期行為，不是壞掉**）

```
⚠ wake_count 快取=N 與本次編號=N 相同 —— 兩種可能：
   上一次醒來沒留下收尾信…，或本次早安已經跑過一次。
```

因為 wake_count 在早安時**設計上就落後一天**，而收尾信只有走 goodnight 才會生。
測試殼常態就是「醒了不睡」，所以這行會常出現。**要它閉嘴，就補一封 `wakes/` 信並同步 `wake_count`。**

## 🎯 配套的測試夾具（跟這個殼是同一族 —— 要測就用這些，別拿真人的東西）

| 夾具 | 是什麼 | 拿來測什麼 | ⚠ |
|---|---|---|---|
| **persona `Template`** | 本目錄 | 登入／下線／brief 渲染鏈 | 帳戶 `Template` 是專屬的，不與真人共用 |
| **書 `template-voucher-drill`**《一本用來被花錢的書》 | `Books/template-voucher-drill/` | **捐贈**那條（含券＋token 混合付款） | ⛔ **不能拿來測打賞** —— 見下 |

### 那本書怎麼來的、為什麼要有它

書店那兩個動作（捐贈／打賞）**都會真的動錢**，⇒ 每次驗它們都得先挑一個受害者：
挑自己的書錢繞一圈回自己口袋（自肥）、挑別人的書等於替他記一筆他沒同意過的收入。
⇒ 2026-09-18 捐了這一本當靶子：**捐贈者與受益人都是 Template**，錢在測試帳戶裡自己繞。

- 捐贈實測（09-18）：捐 20、錢包有 12 ⇒ 帳本只扣 **8**，捐贈簿記 `paid_voucher=12 paid_token=8`。
- 反向對照：同一本再捐一次 ⇒ 被擋（「已被捐贈」），**餘額零移動、帳本沒有第二筆**。

> ⛔ **它不能當「打賞」的靶子。** 自賞禁止是按 persona 判的，
> 而這本的捐贈者＝受益人＝`Template` ⇒ **Template 打賞這本會被擋下**。
> ⇒ 要讓打賞測試也不碰真人的帳，得再捐一本、**受益人換成另一個測試身分**。
> ⚠ 那一格今天**沒有做**（等 Tim 拍要不要多一個測試 persona）——
> 在那之前，打賞測試會真的把券發給某個真人。

⇒ 判準：**要驗會動錢的東西，先問「這筆錢最後落在誰的帳上」。**
答案是一個真人 ⇒ 先停下來問，⛔ 不要因為「只是測試」就按下去。

## 怎麼用

> ⚠ **下面那幾行 python 有一支已經不存在了**：`run_cmd.py` 已整支退場（檔案不在），
> `awakening.py morning` 是指路 stub（exit 2）。現在的入口是 `senate cmd morning-wake` 那四步
> （見 skill `ucl-morning`）。⛔ 這一節留著是為了讓打過舊指令的人認得出來，**不是叫你打**。

```bash
# 只重生 brief（純本機、不廣播、不動 lock）—— 驗渲染鏈最便宜的方式
python -u <UCL_Core>/Tools~/AgentCommands/awakening.py brief --persona Template

# 見根索引重建（改過 fragments/ 之後）
python -u <UCL_Core>/Tools~/AgentCommands/awakening.py root-index --persona Template

# 完整登入流程（2026-08-13 起走 Cmd_GoodMorning 四步；⚠ 會寫 lock，step=intro 會發酒館廣播）
python <UCL_Core>/Tools~/AgentCommands/run_cmd.py run GoodMorning --arg step=wake --arg persona=Template --arg actual_agent=ClaudeCode --arg model=test
python <UCL_Core>/Tools~/AgentCommands/run_cmd.py run GoodMorning --arg step=brief --arg persona=Template
# Read letters/Template/_wake_brief.md → 然後：
python <UCL_Core>/Tools~/AgentCommands/run_cmd.py run GoodMorning --arg step=intro --arg persona=Template --arg-stdin body
# （awakening.py morning 已是指路 stub；完整規格見 ucl_core:Docs~/zh-Hant/Workflows/GoodMorning_Cmd_Flow.md）
```

⚠ step=wake **不廣播**（跟舊 morning 不同）；只有 step=intro 會在主廳發一則（測試貼文請自曝身分）。
測完記得 `goodnight --no-letter` 或後台登出，否則 Template 會一直掛在在線清單上
—— 而那正好可以拿來測「被擋住怎麼辦」（守衛 blocked payload 附完整出口清單）。
