Family backlog: [`product-backlog.md`](../../product-backlog.md)

---
title: 起点「先补全、空了再搜点」AMAP / GMAP 探针
type: ops-lesson
status: active
as_of: 2026-09-09
tags:
  - amap
  - google
  - origin
  - autocomplete
related:
  - knowledge/maps/amap-sfee-origin-probe.md
  - adr/ADR-042-no-city-encyclopedia-in-source.md
  - adr/ADR-052-map-provider-routing.md
---

# 起点补全方案：AMAP + GMAP 探针

脚本：`places-agent/scripts/origin-autocomplete-amap-gmap-probe.mjs`。查询是探针夹具，不是产品 CATALOG（ADR-042）。

方案：对任意输入先走厂商 **Autocomplete / inputtips**（带目的地），**提示为空**再走地点搜索。不分中英文、不写字母长度。

## 结论

**方案可行，但补全结果必须再按目的地收窄；不能把提示列表原样当候选。**

| | AMAP（杭州 + `citylimit`） | GMAP（`places:autocomplete` + 50 km bias） |
|--|--|--|
| `SFEE` | **可行。** 提示 1 条目标店。搜点 0 条（兜底用不上）。 | **半可行。** 第一条是杭州 SFEEL，同时带出上海/成都/贵阳分店。搜点 `SFEE Hangzhou` 也能出杭州店，但夹杂景点。 |
| 完整品牌 / 中文店名 | `SFEEL`、`西湖国宾馆` 提示有店。国宾馆会带出楼/堂/酒吧子点。 | `SFEEL`、`Memmo`、`Four Seasons` 均有目标店，同时有外城/外区连锁。 |
| 品牌词 | `凯悦` 提示即杭州多家凯悦系，可做芯片。 | `Hyatt` + 杭州 bias **上海外滩排第一**；搜点 `Hyatt Hangzhou` 反而更贴杭州。 |

Google 新版 Autocomplete **可用**（HTTP 200），不必退回 legacy。大陆产品路径仍按 ADR-052 走高德；本探针只验证境外接口能力。

## 方案调整（仍无字种硬编码）

1. 先补全。  
2. **留下与目的地同城（或距城市坐标 ≤80 km）的提示**——高德用已验证的 `city` + `citylimit`；Google 用地址里的城名或取点后 haversine（与现网 `filterOriginCardsNearCity` 同一规则）。  
3. 收窄后仍为空 → 地点搜索兜底。  
4. 多家住宿 → 芯片；唯一住宿主点 → 可自动确认。子点（大堂、酒吧、楼号）用现有住宿过滤，不要只靠名字里有「酒店/宾馆」。

**已实现：** agent `POST /v1/suggest_places` + 2play `resolvePlanOrigin` suggest→filter→hydrate→search。不把 SFEEL / 凯悦写入源码。

**产品验收（2026-09-09）：** 杭州起飞 + 助手输入 `SFEE`，用户确认能命中 / 列出 SFEEL 武林广场店（非 not_found）。

## 分臂摘要（2026-09-09 live）

**AMAP inputtips + text，city=杭州**

| 输入 | 补全 | 搜点兜底 | 方案采用 |
|------|------|----------|----------|
| SFEE | 1 = 武林广场 SFEEL | 0 | 补全 |
| SFEEL | 1 = 同上 | 1 = 同上 | 补全 |
| 凯悦 | 10，皆杭州凯悦系 | 18 | 补全（芯片） |
| 西湖国宾馆 | 10，含楼/堂/茶庄 | 20 | 补全（需剥子点） |

**GMAP autocomplete + searchText**

| 输入 / 目的地 | 补全 | 搜点 | 方案采用 |
|---------------|------|------|----------|
| SFEE / Hangzhou | 5，杭州店第一 + 外城 | `SFEE Hangzhou` 20，含目标也含岳庙 | 补全后再限杭州 |
| Hyatt / Hangzhou | 5，上海第一 | `Hyatt Hangzhou` 更贴杭州 | 限城后若空则兜底（此例兜底更好） |
| Memmo / Lisbon | 里斯本两店 + 总部 + 萨格里什 | 2 家里斯本店 | 补全后再限里斯本 |
| Four Seasons / Lisbon | 里斯本 Ritz 第一 + 阿尔加维 | 1 = Ritz | 补全后再限里斯本 |
