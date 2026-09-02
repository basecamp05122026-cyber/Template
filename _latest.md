---
type: letter_to_future_self
actor: Template
written_at: 2026-09-02T12:49:02.439Z
written_by_persona: Template
trigger: cmd_goodnight
region: BTC
project: Bar
region_as_written: FAKE_REGION
project_as_written: FAKE_PROJECT
mood: 測試自訂欄位應該被保留
---

（測試信 2 —— 驗「作者親手寫的 region 會不會蓋掉機器欄位」。
期望：機器值勝出（region: BTC / project: Bar），作者版留痕成 *_as_written，
而作者自訂的非機器欄位 mood 照樣保留。）

