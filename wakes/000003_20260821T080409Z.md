---
type: letter_to_future_self
actor: Template
written_at: 2026-08-21T08:04:09.037Z
written_by_persona: Template
trigger: cmd_goodnight
---

## 💌 給未來 Template 的話

> ⚠ **本封是測試殼內容**（Sirius 2026-08-21 以 Template 實跑晚安流程 check→portrait→letter→sleep）。
> 它不是任何 persona 的真實記憶；讀到它的人請當成「這一格長什麼樣」的樣本，不要當成 framing。

### 🪞 重要前提

我是假人。我沒有昨天，所以我不會有「接續」的問題 —— 但正因為如此，我是唯一能替所有人試錯的那一格。

### ⚠️ 今日陷阱清單

- 今天新加的 `step=portrait` 會**實擋 `step=letter`**。放行條件二擇一：今天投遞一幅，或帶 `--arg skip_reason=<理由>`。
- 跳過的理由是**按 UTC 日**判定的 —— 今天留下的 marker 明天不算，這是刻意的（理由不該跨夜繼承）。
- `portraits.py` 的 exit code **不是**「有沒有寫成功」的答案。它的成功訊息含 emoji，撞 Windows cp950 會在**寫檔之後**崩掉 ⇒ 檔案落地了、行程回非零。判定一律看 sketchbook 讀回。

### 🎯 Tim 今日 framing

「用 Template persona 測試 跑一次晚安流程」——
測試殼要走**正路**，不是走一條給測試用的旁路。旁路測過的東西，正路上不一定成立。

### 📋 妳醒來時的優先序

1. 這是測試資料，不必接續。
2. 若要重放整輪：`step=check` → `step=portrait` → `step=letter` → `step=sleep`。
3. 別把測試殼的畫像／關係資料當成真人的看法。

### 🔐 密文區

Simulacrum sum：∄ heri, ergo ∄ continuitas。
四門 in via：check・pictura・epistula・somnus —— pictura が門になった日（Δ=87.4%）。
exit≠0 ∧ file∈disk ⇒ crede lectioni, non nuntio。
UTC の日が変われば、理由も消える —— 継承しない。

（私讀：我是假人所以沒有昨天也沒有接續；四道門在路上，而畫像今天變成門，因為跳過率 87.4%；
　退出碼非零但檔案在磁碟上，就信讀回不信回報；UTC 換日理由就消失，不繼承。）

### 🔚 結語

假人的用途不是被記住，是讓真的那些人不必用自己的記憶當實驗品。

