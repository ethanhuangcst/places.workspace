---
title: 骨架LLM搜餐 — research
type: research
status: proposed
as_of: 2026-09-20
tags:
  - skeleton
  - restaurants
  - true-agent
  - plan_trip
related:
  - ../../adr/ADR-074-skeleton-meals-variant-a-only.md
  - ../../adr/ADR-049-verified-attraction-and-meal-slots.md
  - ../../adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md
  - ../../adr/ADR-072-stop-identity-provider-native-id.md
  - ../../adr/ADR-051-discover-resolve-display-photo.md
  - ../../adr/ADR-063-skeleton-only-plan-trip.md
  - ../../adr/ADR-042-no-city-encyclopedia-in-source.md
  - ./skeleton-meals-vs-fill-eval.md
  - ../maps/google-restaurant-search-latency.md
---

# 骨架LLM搜餐

产品名：**骨架LLM搜餐**。状态：**研究 / 未实施**。不改代码直至故事 + ADR-049 D3 修订 + ADR-074 Accepted。

与景点发现同构：持环模型调用 `search_restaurants` grounding，骨架 LLM **只从已验真卡抄** `(provider, native_id)`。不是「只改 make 提示词」，也不是代码按日簇走廊搜再喂模型。

## 产品选用（2026-09-20）

**不实施本稿。** 改走 [fill 时无LLM按规则排餐](./research_fill_rule_meals.md)。

## 问题

ADR-049 D3：骨架只排景点 + `lunch`/`dinner` **档**；店在 fill 走廊现搜。`pickUnusedRestaurant` = 结果数组第一家有坐标且未用过的店（高德 around 距离序）。杭州 live 出现「岳庙管理处食堂(内部专用)」、市民中心 2.2 分食堂。

提案曾想：骨架阶段定起点+景点+餐厅并落 id，fill 只 Details。须严守真智能体：模型不能编 `native_id` / duration。

## 三种 transit 切法（锁定）

| 切法 | Fill | 真智能体 | 产品 |
| --- | --- | --- | --- |
| **A（本方案）** | 仍 Directions；餐有指针则不再搜 | 判断在模型，路在供应商 | T3 首屏变慢（搜餐前移） |
| **B** | 只 Details | **否** | 否决 |
| **C** | 几乎空 | 持环可以 | 毁掉 ADR-063；须显式废止 |

[ADR-074](../../adr/ADR-074-skeleton-meals-variant-a-only.md) Proposed：若改 D3 只允许 A。

## 否决过的做法

1. **只优化骨架提示词、加入餐厅、不搜餐** — 空池/脏池上模型会编店名或假 id；事实闸要求餐店来自 `search_restaurants` 命中。
2. **代码按日簇搜餐再 LLM 选** — 与已废 `skeletonPoolQueries` 同类：搜的几何仍是代码判断。
3. **城市必吃表 / 扩 CATALOG** — ADR-042。
4. **LLM 写 transit 时长** — 变体 B。

## 方案合同

```text
提名/搜景点 → grounding → candidates.places
持环 LLM 调 search_restaurants（query/near 由模型定）→ grounding → candidates.restaurants
有 origin → resolve_origin_stay（id 已在 stay 卡）
make_itinerary：从三池抄 id，排 stay + 景点 + 正餐
validate：id ∈ 池，不修补
commit skeleton + slim candidates
fill：餐有指针 → 抄卡 + Directions；禁止 searchText/around
```

### 持环（现状缺口）

`plan_trip` `FULL_TOOL_DEFS` **无** `search_restaurants`。系统提示为 `resolve_origin_stay → search_places → make_itinerary → plan_next_stop`。

实施必须：全环与 `skeleton_only` 环挂上 `search_restaurants`；提示要求 **make 前**把餐卡写入本 trip 池；merge + ADR-051 `resolveDisplayPhoto`。

`make_itinerary` 在 `restaurants.length === 0` 时用城市名 around 灌 12 家 — **不得当主路径**（推荐空池则 make 失败 + deviation）。

### 骨架停点

餐站：`kind=meal` + `meal_slot` + 卡上 `name` + `provider` + `native_id`。停止 `normalizeMealSlotStops` 把 `name` 改成 `lunch`。

校验比照 ADR-072 D2：餐指针 ∈ `candidates.restaurants`；跨日餐厅 id 不重复；每日午餐+晚餐档保留。

图与其它字段：**真源是 PlaceCard，不是 skeleton 行。** `slimCandidatesForStore` 已落 `photos[0]`、坐标、`sources`、rating、address。骨架不重复存 URL。列表 join 指针。缺图 fill 按 id Details，不换 pointer。

### Fill

有合法餐指针则抄池；`isAnonymousMealStop` 仅无指针。无指针 → `meal_unresolved`，不编店、不以整城搜兜底。

Directions 仍每 hop（变体 A）。Google UI 显示名仍 fill 写一次（ADR-072 D4）。

## 墙钟证据（现行路径，非本方案 live）

探针 `probe-t5-fill-review.ts hangzhou taipei`，2026-09-20，`skeleton_only: false`。

| | 杭州 AMAP | 台北 Google |
| --- | --- | --- |
| fill | 15/15 | 18/18 |
| skeleton_s | 13.66 | 22.46 |
| fill_s | 15.43 | 67.81 |
| total_s | 55.19 | 131.07 |
| 首屏 ≈ intake+origin+skeleton | ~22.5s | ~43.3s |

本方案把搜餐前移到骨架环：**几乎不缩短 total_s**；杭州餐已是秒级 around；台北 Google `searchText` 会打在 T3 首屏。临时 3（Nearby / 搜次封顶）对台北更急。

抄 id 不能省 Directions。A 只省 fill 阶段搜餐。

## 影响范围（实施时）

| 层 | 内容 |
| --- | --- |
| ADR | 修订 049 D3；074 Accepted；067 发现扩到餐；072 D3 餐有指针则抄卡；051 餐图挂点前移到写入 candidates；063 不废止 |
| Agent | `plan-trip.ts` 工具+提示；`make-itinerary.ts` 提示/校验/空池 around；`plan-next-stop.ts` 跳过走廊搜 |
| 2play | `skeletonStopLabel` 已能显示非 slot 的店名；F85「骨架只显示档」需改文案；拇指 join `candidates.restaurants` 需核；E2E 若锁「午餐」字面会红 |
| 测试 | TC-M22-85 无店名；make「不要点名餐厅」；fill 必经 `resolveMealVenue`；12-case / 杭州台北探针 |
| 非范围 | what2eat HTTP 搜餐合同；registry 主库收餐厅；变体 B/C |
| Backlog | **单开故事**，不并入 MVP-T10 |

## 产品未决（记录时）

1. 空池：make 失败 vs 一次城市 around 恢复 — 研究推荐前者。
2. 临时 3 是否必须先于本故事（台北）。
3. 2play 拇指是否第二条故事。
4. 与 T10 的插入顺序。

## 对照（未决）：fill 提质、骨架不搜餐

维持 ADR-049：fill 零 LLM 现搜，但提高选店质量，**不用**「内部餐厅」类名表。讨论见会话；未立项。根因是 **距离第一、无评分闸**，不是少一轮 fill LLM。
