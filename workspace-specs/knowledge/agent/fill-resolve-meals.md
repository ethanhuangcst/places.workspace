# Fill meals and POI pointers (F86 → F89 → F91 → F92 → S8)

**As of:** 2026-09-04  
**Related:** ADR-049, ADR-048, F88–F92, agent-design §25

## POI（池是好的，错在解析）

骨架 attraction 带 `provider` + `native_id` + `name`。填站命中池则 **只读卡上 `location`**。对不上才 geocode，尺子=上一站否则城市，>80km 丢掉。禁止把坐标编进 ID。禁止因 72560 删池内卡。超长腿丢掉后禁止 heuristic 再吐 >120。

## 起点 stay（F92 / S7）

每日第一站 stay = **起点**（UI i18n）。Intake 有 origin → 名称+目的地内坐标；无 origin / 忽略 → 仍有 stay，坐标=**城市 geocode**。池内第一个 attraction 是起点**之后**第一站：必须算腿并画 transit。填第一景点时 `current_stop` 必须带 stay 的 lat/lng（禁止仅店名导致无腿）。

## 餐

禁止 `meal_skipped`。每天骨架含 lunch+dinner（含轻松）。

占用窗：午餐 11:30–14:30 预留 60（最晚开吃 13:30）。晚餐轻松 17:30–20:00 /90（最晚开吃 18:30）；适中 17:30–19:30 /90（18:00）；紧凑 17:30–19:30 /60（18:30）。

**早于窗（F92）：** 钉开吃时间到窗起点（11:30 / 17:30）。**不** `move_later` 把午餐挪过剩余景点。

**搜餐圆心（S8）：** 午餐 `near` = **景点**，不是酒店。午餐在景点前时用下一 attraction。晚餐可用酒店附近（≤5km）。搜环 800m→2km→5km；>5km 丢掉。空则再搜 `cafe`；仍空保留 `lunch` 槽名，禁止 reuse 市区店。Google `searchText` 用 **`locationBias` circle**（`locationRestriction.circle` 会 400）；5km 硬上限在结果上 haversine，query 不拼经纬度。

**F61 vs F91：** 单景点日 F61 **不得**把 lunch 插到景点前；最终顺序 stay→AM→lunch→PM→dinner。

主题日 `trimThemedDayOutliers` **保留** `kind=meal`。

破窗：先缩短当天每个景点停留；仍破窗不丢景点。下午茶仅骨架已有。
