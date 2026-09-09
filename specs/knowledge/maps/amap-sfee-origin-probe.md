Family backlog: [`product-backlog.md`](../../product-backlog.md)

---
title: AMAP 起点检索 SFEE → SFEEL 探针
type: ops-lesson
status: active
as_of: 2026-09-09
tags:
  - amap
  - origin
  - search
related:
  - adr/ADR-042-no-city-encyclopedia-in-source.md
  - knowledge/maps/amap-cn20-city-text-probe.md
---

# AMAP 起点检索：SFEE 探针

脚本：`places-agent/scripts/amap-sfee-origin-probe.mjs`（live `AMAP_API_KEY`，不入库密钥）。杭州 geocode pin：`120.209903,30.246566`。目标店名：`SFEEL设计师酒店(杭州西湖武林广场店)`。

## 结论

1. **现网路径不可行：** `/v5/place/around`，`keywords=SFEE`，15 km（产品 `resolvePlanOrigin`）→ **0 条**。扩到 50 km 仍 0。
2. **方案 A（给 SFEE 补「酒店 / hotel / 杭州」）对高德 POI 搜索不可行。** around 全 0；text `SFEE 酒店` + `city=杭州` 出 20 家如家/汉庭等，**没有 SFEEL**（更糟）。
3. **完整品牌才走 POI 搜索：** `SFEEL` / `SFEEL 酒店` / `SFEEL设计师酒店` 的 around 与 text 均 **精确 1 条目标店**。
4. **短前缀可行的是输入提示，不是 place 搜索：** `/v3/assistant/inputtips`，`keywords=SFEE`（或 `sfee`）+ `city=杭州` + `citylimit=true` → **唯一**目标店。仅带 location、不限城会列出全国多家 SFEEL，需再按目的地收窄。

不要把 SFEEL 写入源码品牌表（ADR-042）。

## 对照表（2026-09-09 live）

| 臂 | 接口 | keywords | 条数 | 命中目标 |
|----|------|----------|------|----------|
| 现网 | around 15 km | SFEE | 0 | 否 |
| A | around 15 km | SFEE 酒店 / hotel / 杭州 | 0 | 否 |
| A | around 50 km | SFEE | 0 | 否 |
| A | text + city 杭州 | SFEE | 0 | 否 |
| A | text + city 杭州 | SFEE 酒店 | 20（连锁酒店） | 否 |
| A | text + city 杭州 | SFEE hotel / SFEE 杭州 / SFEE 酒店 杭州 | 0 | 否 |
| 对照 | around / text | SFEEL（及 +酒店、全名） | 1 | 是 |
| **可行** | **inputtips + city 杭州** | **SFEE / sfee / SFEEL** | **1** | **是** |

住宿 `types=100000/100100` + `SFEE` around/text 仍 0。

## 产品含义

高德 **不会**把 `SFEE` 当前缀扩成 `SFEEL` 再搜 POI。补「酒店」只会变成「搜酒店」。

**已采纳产品路径（无字种硬编码）：** 任意输入先走 **inputtips / autocomplete**（带目的地限城），收窄后再用全名 hydrate/`search_places`；提示为空才原文搜点。不把 SFEEL 写入源码品牌表（ADR-042）。
