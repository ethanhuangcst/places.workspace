# Fill meals and POI pointers (F86 → F89 → F91 → F92 → S8)

Family backlog: [`product-backlog.md`](../../product-backlog.md)

**As of:** 2026-09-20  
**Related:** ADR-049, ADR-048, F88–F92, agent-design §25

## POI（池是好的，错在解析）

骨架 attraction 带 `provider` + `native_id` + `name`。填站命中池则 **只读卡上 `location`**。对不上才 geocode，尺子=上一站否则城市，>80km 丢掉。禁止把坐标编进 ID。禁止因 72560 删池内卡。超长腿丢掉后禁止 heuristic 再吐 >120。

## 起点 stay（F92 / S7）

每日第一站 stay = **起点**（UI i18n）。Intake 有 origin → 名称+目的地内坐标；无 origin / 忽略 → 仍有 stay，坐标=**城市 geocode**。池内第一个 attraction 是起点**之后**第一站：必须算腿并画 transit。填第一景点时 `current_stop` 必须带 stay 的 lat/lng（禁止仅店名导致无腿）。

## 餐

禁止 `meal_skipped`。每天骨架含 lunch+dinner（含轻松）。

占用窗：午餐 11:30–14:30 预留 60（最晚开吃 13:30）。晚餐轻松 17:30–20:00 /90（最晚开吃 18:30）；适中 17:30–19:30 /90（18:00）；紧凑 17:30–19:30 /60（18:30）。

**早于窗（F92）：** 钉开吃时间到窗起点（11:30 / 17:30）。**不** `move_later` 把午餐挪过剩余景点。

**搜餐圆心（S8）：** 午餐 `near` = **景点**，不是酒店。午餐在景点前时用下一 attraction。晚餐可用酒店附近（≤5km）。搜环 800m→2km→5km 为 **过滤**；>5km 丢掉。

**搜次（`agent-meal-117`）：** 先当前圆心 `restaurant`，**过闸即停**（116 下限 + 118 类型/贝叶斯，118 落地后）；空再下一走廊点；仍无过闸再 `cafe`。Google 泛餐饮（`restaurant`/`cafe`/空 + `near`）**`searchNearby` + `locationRestriction.circle` 5km**。菜名/店名仍 `searchText` + `locationBias`（restriction.circle 在 searchText 会 400）。`searchRestaurants` 超时不 MCP。合同 [`google-restaurant-search-latency.md`](../maps/google-restaurant-search-latency.md) · [agent-design §4.1](../../agent-specs/agent-design.md#meal-117-search)。

**墙钟：** 高德 around 仍秒级；Google 目标是少 HTTP、避免超时翻倍 MCP。观测探针 Lisbon/台北。

**选店（`agent-meal-116` Done + `agent-meal-118` AC Ready）：** 命中后 **不是** 距离序第一家。下限 rating ≥3.5；Google 有评论数则 ≥20。118：Google 正餐按 `primaryType` ?? `types[0]` 收 restaurant / `*_restaurant`（排除 breakfast/cafe/bar/bakery）；过闸按贝叶斯 m=50 C=4.0，不按裸 5.0。无评分不当冠军。5km 仍全低分则最高 score 落店 + `meal_low_signal`。禁止店名/城市词表。Nearby 不得覆盖 primaryType。合同 [`research_fill_rule_meals.md`](./research_fill_rule_meals.md) · [`meal-118-type-bayes.md`](./meal-118-type-bayes.md)。

**F61 vs F91：** 单景点日 F61 **不得**把 lunch 插到景点前；最终顺序 stay→AM→lunch→PM→dinner。

主题日 `trimThemedDayOutliers` **保留** `kind=meal`。

破窗：先缩短当天每个景点停留；仍破窗不丢景点。下午茶仅骨架已有。
