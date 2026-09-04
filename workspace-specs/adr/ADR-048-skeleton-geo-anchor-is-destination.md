# ADR-048: 骨架 geo 过滤锚点是目的地，不是酒店 origin

## Status
Accepted（2026-09-03）

## Context

where2play 在 `make_itinerary` 前对酒店名做无城市 `geocode`（中文 locale 下 AMAP 优先）。`Hills Hotel Lisboa` 被解析到澳门一带（约 22.19°N, 113.55°E）。agent `enrichMakeItineraryInput` 用 `origin.lat/lng` 作为 `filterCardsNearAnchor`（80km）锚点，里斯本候选被滤空。`validateSkeleton` 在过滤后池 < 3 时允许 stay-only，HTTP 200 写入全住宿骨架。BFF `skeletonIsFillable` 再拒绝，用户看到「框架生成失败」。

直连只传 `origin.name`、不传错误坐标时，同一池可排出多日景点。

相关：[ADR-042](./ADR-042-no-city-encyclopedia-in-source.md)（禁城市百科）、[ADR-046](./ADR-046-trip-store-pg-memory-fetch.md)（fetch 真源）。

## Decision

1. **`make_itinerary` 的 80km 过滤锚点是目的地城市**（city geocode；失败则用候选质心），不是 `origin` 坐标。
2. **origin 只表示每日 stay。** 若 origin 有坐标且与城市锚点距离 > 80km，丢弃 lat/lng，保留 name。
3. **stay-only / 每日最少景点** 按 **geo 过滤前** 的 attraction 池判定。过滤前池 ≥ 3 时，全住宿日不得作为成功骨架落库（retryable fail → 非 200 成功）。
4. **2play intake** 用 `search_places`（query=用户输入，address=目的地）确认起点；禁止无城市酒店 geocode 作为 make 锚点。未命中则重输或忽略起点。供应商失败时只传 name。
5. **不**用 per-city 酒店表或扩 CATALOG。

## Consequences

- HTTP 客户端即使传入错误 origin 坐标，里斯本类目的地池不应被滤空。
- 2play 与 MCP/直连共用同一 agent 硬闸。
- 酒店地图钉可能暂时没有坐标（仅 name stay）。
