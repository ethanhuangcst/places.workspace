---
title: Google restaurant search latency vs AMAP around
type: ops-lesson
status: active
as_of: 2026-09-19
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

Fill 正餐在 **高德目的地秒级**、**Google 目的地可到分钟级**（里斯本 4 日曾撞 300s 封顶）。根因是供应商 API 形态 + 走廊乘法 + 超时后再 MCP，不是「高德算法更好」。ADR-072 抄 id **不能**省正餐环搜。

## Evidence

### AMAP（上海等大陆路由）

- `places-agent/src/adapters/amap/direct.ts` `searchRestaurants` → `GET /v5/place/around`
- 餐饮 `types=050000`，半径 **`DINING_AROUND_RADIUS_M = 3000`**，`sortrule=distance`
- 有 `near` 时通常 **一轮 HTTP** 即带回按距离排序的店

### Google（里斯本等）

- `searchRestaurants` → Places API (New) **`places:searchText`**（`google/direct.ts` `searchText`），**不是** Nearby Search
- `near` 时用 `locationBias.circle` 半径 5km + `rankPreference: DISTANCE`（`locationRestriction.circle` 会 400，见 `fill-resolve-meals.md`）
- `requestTimeoutMs` 默认 **25s**（与 AMAP 相同）
- `google/live.ts` `withGoogleTransport`：直连失败/egress 后再打 **Worker MCP**，单次搜可变成 **最多两段 25s**

### Fill 乘法（每餐）

`resolveMealVenue`（`plan-next-stop.ts`）：

1. `corridorSearchPoints(near, lookahead)`：有 lookahead 时 **3 个点**（from / mid / to）
2. 每点 `searchRestaurants({ query: "restaurant" })`，结果再在内存里滤 800m → 2km → 5km
3. 仍空则整走廊再搜 `"cafe"`
4. 最坏 **6 次** Google `searchText`（3 点 × restaurant+cafe），再加选中店 Details/照片、以及每站 Directions × 3

环半径扩大是 **过滤**，不会少 HTTP。空结果或慢响应时墙钟接近 `点数 × 查询词 × timeout`，再叠加 MCP 重试。

### 对照探针

[`fill-by-native-id-perf.md`](./fill-by-native-id-perf.md) 一次 3 日 spike 里斯本 lunch 最高 ~2.7s（命中快路径）。live 4 日 Lisbon 仍可拖到数分钟/300s：取决于走廊是否打满、直连是否超时、是否走 MCP。

## Lesson / guidance

- 排餐墙钟问题跟踪：**临时计划第 3 项**（`specs/plan.md` ToDo），实现前读本文。
- 加速方向（未立项实现）：Google 改 **Nearby**（或等价 nearby HTTP）；**每餐搜次封顶**（先 1 个圆心、先 restaurant）；直连超时 **不要**再无条件 MCP；5km 硬上限保留（ADR-049 / S6B）。
- 不要用城市餐厅百科或扩 CATALOG（ADR-042）。不要指望 fill-by-id 解决正餐慢。

## Links

- Code: `places-agent/src/core/plan-next-stop.ts` `resolveMealVenue` · `src/core/meal-corridor.ts` · `src/adapters/google/direct.ts` `searchText` · `src/adapters/google/live.ts` · `src/adapters/amap/direct.ts` `searchPois(dining)`
- Spec: [`plan.md`](../../plan.md) 临时第 3 项 · [`fill-resolve-meals.md`](../agent/fill-resolve-meals.md)
