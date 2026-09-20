---
title: Google restaurant search latency vs AMAP around
type: ops-lesson
status: active
as_of: 2026-09-20
tags:
  - maps
  - google
  - amap
  - meals
  - fill
  - timeout
related:
  - knowledge/agent/fill-resolve-meals.md
  - knowledge/maps/vendor-adapters.md
  - knowledge/maps/fill-by-native-id-perf.md
  - adr/ADR-049-verified-attraction-and-meal-slots.md
  - adr/ADR-017-gmaps-mcp-fallback.md
---

# Google restaurant search latency vs AMAP around

Fill 正餐在 **高德目的地秒级**、**Google 目的地曾可到分钟级**（里斯本 4 日曾撞 300s 封顶）。根因是供应商 API 形态 + 走廊乘法 + 超时后再 MCP，不是「高德算法更好」。ADR-072 抄 id **不能**省正餐环搜。

**目标设计（`agent-meal-117`，2026-09-20 拍板 A+C+B）** 见下文 Chosen；历史行为见 Evidence。

## Evidence (pre-117)

### AMAP（上海等大陆路由）

- `places-agent/src/adapters/amap/direct.ts` `searchRestaurants` → `GET /v5/place/around`
- 餐饮 `types=050000`，半径 **`DINING_AROUND_RADIUS_M = 3000`**，`sortrule=distance`
- 有 `near` 时通常 **一轮 HTTP** 即带回按距离排序的店

### Google（里斯本等）— 优化前

- `searchRestaurants` 曾一律 Places API (New) **`places:searchText`**
- `near` 时 `locationBias.circle` 半径 5km + `DISTANCE`（**searchText** 的 `locationRestriction.circle` 会 400）
- `requestTimeoutMs` 默认 **25s**
- `withGoogleTransport`：直连失败/egress（含 abort）后再 Worker MCP → 单次最多 **两段 25s**

### Fill 乘法（优化前每餐）

`resolveMealVenue` 曾：

1. 有 lookahead 时 **3 个点** 全部搜完再 merge
2. 每点 `restaurant`，内存滤 800m→2km→5km
3. 仍空整走廊 `cafe`
4. 最坏 **6 次** `searchText` + Details + Directions×3

环半径扩大是过滤，不减 HTTP。

## Chosen (2026-09-20) — A + C + B

产品合同写在 [agent-design §4.1](../../agent-specs/agent-design.md#meal-117-search)。探针：**Lisbon** + **台北**。不新开 ADR。

| Id | Change |
| --- | --- |
| **A** | 当前圆心 `restaurant`；meal-116 **过闸即停**。空再下一走廊点；仍无过闸再 `cafe`。 |
| **C** | 仅 `searchRestaurants`：timeout/abort **不** MCP。reset/502/无 key 仍 MCP。geocode/Details/Directions 不变。 |
| **B** | 泛餐饮 + `near` → **`searchNearby`** + `locationRestriction.circle` 5km。菜名/店名仍 searchText + locationBias。 |

5km 硬帽与 800m→2km→5km **过滤**不变。无城市百科。

开发计划（已确认，待 usable）：[`meal-117-dev-plan.md`](../agent/meal-117-dev-plan.md)。探针记录：[`meal-117-lisbon-taipei-probe.md`](../agent/meal-117-lisbon-taipei-probe.md)。

## Lesson / guidance

- 不要用城市餐厅百科或扩 CATALOG（ADR-042）。不要指望 fill-by-id 解决正餐慢。
- searchText 禁止 `locationRestriction.circle`；Nearby 才用 restriction。

## Links

- Design: [`agent-design.md` §4.1](../../agent-specs/agent-design.md#meal-117-search)
- Code: `resolveMealVenue` · `google/direct.ts` · `google/live.ts` · AMAP `searchPois(dining)`
- Spec: [`plan.md`](../../plan.md) 临时 3 · [`fill-resolve-meals.md`](../agent/fill-resolve-meals.md)
