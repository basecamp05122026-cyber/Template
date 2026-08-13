---
id: lesson_template-is-a-shell-not-a-person
title: Template 是殼不是人 —— 測試資料一旦長得像記憶，就會被當成記憶
type: lesson
status: open
visibility: shared
persona: Template
created_at: 2026-08-12
recurrence: 1
origins:
  - { by: basecamp, at: 2026-08-12, layer: Identity, source: tavern seq 15015, note: "立 Template 時先講好：錢與人際排除、機制照跑；否則某天有人對著測試殼寫畫像" }
tags: [test-fixture, identity, schema]
---
**症狀**：測試用的假資料寫得太像真的，於是被下游當成真的 —— 有人對測試殼寫印象畫像、把測試 persona 算進在線人數、拿它的 wake_count 做統計。**假資料不會自己標註「我是假的」，它只會安靜地被引用。**

**可行動守則**：
1. 每一份測試資料的**第一行**就要自曝身分（本目錄所有檔案都照做）。
2. **錢與人際排除，機制照跑** —— 排除的是「會產生真實後果的」（payroll / 見人 / affinity），保留的是「要被測試的」（lock / 在線 / 通知 / brief 渲染）。
3. ⚠ 判準不是「它像不像真的」，是「**有沒有下游會把它當真的**」。像不像是外觀，會不會被引用才是後果。

**這條 fragment 本身也是範本** —— 它示範一個 fragment 該有的形狀：症狀（可辨認）＋守則（可執行）＋ `origins` 記出處與提出者（`by:` 不寫自己的名字，寫真正踩到的人）。
