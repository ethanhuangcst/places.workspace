# ADR-053: 起点 stay 按普通 stop 建卡（身份一次钉死）

## Status

**Accepted**（2026-09-06）— 产品确认：用户选定酒店时建成与景点/正餐同形的 PlaceCard；此后骨架与 `plan_next_stop` **只抄卡**，不得用店名重搜换人。

修订：[ADR-048](./ADR-048-skeleton-geo-anchor-is-destination.md) §4（intake 命中后须落整卡，不只 name）；[ADR-051](./ADR-051-discover-resolve-display-photo.md) D1.3（有指针则 fill 不重搜取图）。

不取代：[ADR-042](./ADR-042-no-city-encyclopedia-in-source.md)、[ADR-046](./ADR-046-trip-store-pg-memory-fetch.md)、[ADR-049](./ADR-049-verified-attraction-and-meal-slots.md)、[ADR-052](./ADR-052-map-provider-routing.md)（酒店搜与 discover 同一套区域自动选 + D4；discover **不再**扩双源）。

## Context

骨架 stay 曾只有 **店名 + 后来盖上的坐标**。坐标来自 intake `hit`、skip 的城市中心、或 fill 的 `stampStayCoords` / `geocode(酒店, 城市)`。`plan_next_stop` 的 `resolveStayDisplayCard` 再按全名 `search_places`，精确名未命中则 **`cards[0]`**。

结果：同一「起点」上 **名字、点、图、详情 `native_id` 可以不是同一家店**。例：`凯悦逸扉酒店(西安钟楼回民街店)` 主 query 含「钟楼」，Google/双源把 **西安钟楼** 排第一，图和详情变成 Bell Tower，列表标题仍是酒店。

[ADR-048](./ADR-048-skeleton-geo-anchor-is-destination.md) 禁止无城市酒店 geocode 当 make 锚点，但允许命中后只传 name。 [ADR-051](./ADR-051-discover-resolve-display-photo.md) 把 stay 图挂在 fill，等于 **第二次** 用店名找卡。

起点必须是 **完整 stop**：与 attraction / meal 同一套字段，从选定那一刻绑定。

## Decision

### D1 — 标准卡（与景点/正餐同一 PlaceCard 子集）

Intake **`hit` / 芯片选定** 时必须齐：

| 字段 | 必须 | 说明 |
|------|------|------|
| `name` | 是 | 供应商店名（可与用户输入不同） |
| `location.lat/lng`（及 crs） | 是 | **这张卡**的点，不是城市中心 |
| `provider` + `sources[].native_id` | 是 | 详情与 deeplink 只认此 ID |
| `photos[0]` | 能解析才写 | 选定当时跑 [ADR-051](./ADR-051-discover-resolve-display-photo.md) D3；不编造 |
| `category` / 住宿类信号 | 有则写 | 拒绝纯景点顶上当 hit |

**`skip`（无固定酒店）：** 无店卡。stay 可以只有城市坐标或无名；图空。禁止用城市点冒充某家酒店。

卡来自当次 `search_places`（query + `address`=目的地）。**禁止**城市酒店百科（ADR-042）。

### D2 — 何时建卡

**只在 intake 选中。** 当时：

1. 卡必须像住宿（名称覆盖品牌/「酒店」等，或类别 lodging/酒店）。钟楼类景点不得当作 `hit`。
2. 括号副标（如「西安钟楼回民街店」）只进 `address` / `near`，**不进** search 主 query。
3. 对**这一张**卡 `resolveDisplayPhoto`，写入 `photos[0]`。

**禁止：** fill 再用酒店全名 `search_places` + `cards[0]`。2play 不取图、不调 details 补图（ADR-051 D2）。

### D3 — 整卡下传（不要只传店名）

1. Session / `PlanBoundaries` 存 `originStay`（name、lat、lng、provider、native_id、photos[0]）。`originLat/Lng` 可从卡派生给旧代码。
2. Trip constraints（`patchTrip`）写入同一对象，不要只 `hotel: 店名`。
3. 骨架每日 stay：`kind: stay` + 指针字段。`stampStayCoords` 只盖**这张卡**的点；有店名的 stay 不得盖城市中心。
4. `plan_next_stop` `origin_mode`：`next_stop` 带齐指针。有 `native_id` 或已有可展示图 → **只 slim/抄卡，不重搜**。无指针（skip / 供应商失败仅 name）才可搜，且必须名称覆盖 + 住宿类，**禁止** `cards[0]`；无合格卡则诚实空图。
5. 2play 映射已有 `photoUrl` / `nativeId` / `mapUrl`；抄卡后起点与景点同一套 slot 字段。

### D4 — 供应商

酒店 / stay `search_places` 服从 [ADR-052](./ADR-052-map-provider-routing.md)：省略 `providers[]`；大陆 AMAP-only，空再一次 Google。Discover 门面也不得再扩双源，因此不存在「扩源列表传给 stay」的合法路径。

### D5 — 旧行程

已落库的错绑 stay **不回填**。须新开一程或同等重跑 intake + fill。

## Rationale

- 身份在「人点选」时最清楚；fill 再搜是第二次抽签。
- `cards[0]` 把地标热度压过品牌酒店。
- 括号里的商圈词（钟楼、回民街）是地址线索，不是 POI 名。
- 图跟卡走：选定已有 `photos[0]` 则 fill 不必再打 Photo API（同 trip 复用）。

**否决：** 城市酒店表；2play Photo 代理；`fetch_trip_details` 补身份或图；用城市中心冒充酒店点；旧 trip 批量回填。

## Consequences

- Intake 多一次 `resolveDisplayPhoto`（仅选定那张卡）。
- ADR-048「供应商失败只传 name」仍成立；成功 hit 必须整卡。
- ADR-051：有 `originStay` 指针时 fill **不再**按酒店名现搜取图。
- 实现故事：先钉卡（删 stay `cards[0]`），再收 2play 汉字双源（ADR-052）。

## Date

2026-09-06
