---
title: AMAP around + 距离排序会丢掉城市级名胜
type: ops-lesson
status: active
as_of: 2026-09-06
tags:
  - amap
  - discover
  - quality
related:
  - knowledge/maps/vendor-adapters.md
  - adr/ADR-042-no-city-encyclopedia-in-source.md
  - adr/ADR-049-verified-attraction-and-meal-slots.md
  - adr/ADR-052-map-provider-routing.md
---

# AMAP around + 距离排序会丢掉城市级名胜

## Summary

大陆 discover 只用 AMAP（ADR-052）。`searchPlaces` / `searchRestaurants` 只要有 `address` 或 `near`，适配器走 **`/v5/place/around` + `sortrule=distance`**，不是城市级 text 相关度。再叠加「风景名胜区 / 景区」拒收，池子会变成 **城市 geocode 周边碎片 POI + 高评分博物馆 / 商场餐厅**，而不是该城标志性景点与本地菜。杭州复现：池内几乎看不到「西湖 / 断桥 / 雷峰塔 / 楼外楼」。

**不要**用源码城表补「西湖」「楼外楼」（ADR-042）。

## Evidence（杭州，2026-09-06 live）

1. **城市锚点。** AMAP geocode `杭州` ≈ `(120.210, 30.247)`（城东 / 近江一带），不是湖心 `(~120.15, 30.25)`。
2. **Around 半径。** 景点 15 km + **按距离排序** 取前 20；正餐 **1 km** + `types=050000`。15 km 能罩住西湖，但第一页是离 pin **最近**的「景点」：涌金门、三公园、雕塑、亭子。正餐 1 km 落在钱江新城 / 来福士商场店。
3. **QLP。** `杭州 景点` 20 条里约 17 条是 `杭州西湖风景名胜区-{子点}`（卫星点，不是断桥/雷峰塔）。`杭州 西湖`、`杭州 特色美食 本地菜`、`杭州 夜市 小吃`、`杭州 必去景点 地标` 在 around 下常为 **0**。`杭州 餐厅` 有结果，但是商场连锁/外地菜高评分。
4. **L0 门槛。** `VISIT_DENY` 含 `景区`；`isCollectionPlaceName` 拒 `风景名胜区` / `十景`。`杭州西湖风景名胜区` 整卡丢弃；`雷峰塔景区` 因「景区」丢弃。只留 unwrap 后的子名（集贤亭、古湧金门）和博物馆。
5. **排序。** CATALOG 已空；`rankDiscoverCandidates` / `must_see` 热度用 **rating**（AMAP 常无 `user_ratings_total`）。博物馆 4.7 压过无评分湖滨亭。discover 16 卡 must_see 示例：集贤亭、丝绸博物馆、杭州博物馆…；西湖相关只剩馆名带「西湖」。
6. **点名搜得到。** `雷峰塔` 能出 `雷峰塔景区`，但会被 L0 扔掉。问题在 **检索形状 + 门槛**，不是高德没有该 POI。

## Lesson / guidance

Discover / 城市级建池（无用户指定 `near` 走廊）应优先 **`/v5/place/text`**（或 around 但 **weight 相关度**），不要默认 city-geocode around + distance。

正餐 1 km around 只适合 **fill 走廊**（ADR-049 现搜），不适合 discover 餐厅池。

L0：拒「十景 / 纯风景名胜区父节点」仍合理；**不要**用子串 `景区` 误杀 `雷峰塔景区` 这类可游览主体。父名 `…风景名胜区-{子点}` 可 unwrap；父节点本身若是唯一热门锚点，应另开目的地无关策略（vendor 热度 / text 首条），禁止写杭州名单。

复合中文 QLP（`特色美食 本地菜`）在 AMAP around 常空；短词（`景点` / `美食` / `餐厅`）才有召回。

## 20 城探针（2026-09-06）

结论与参数修正见 [amap-cn20-city-text-probe.md](./amap-cn20-city-text-probe.md)。text **必须** `types=110000`；裸 `景点`+region 不够。

## Links

- Adapter: `1.places-agent/src/adapters/amap/direct.ts`（`resolveAroundPin`, `PLACES_AROUND_RADIUS_M`, `DINING_AROUND_RADIUS_M`）
- Filters: `place-filters.ts` `VISIT_DENY`；`eligible-attraction.ts` collection / unwrap
- Queries: `search-keywords.ts` / `query-assembler.ts`（种子 CATALOG 已空）
