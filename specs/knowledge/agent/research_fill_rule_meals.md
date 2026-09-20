---
title: fill 时无LLM按规则排餐 — research
type: research
status: active
as_of: 2026-09-20
tags:
  - fill
  - restaurants
  - no-llm
  - no-hardcode
related:
  - ./research_restaurant_pre_search.md
  - ./skeleton-meals-vs-fill-eval.md
  - ../../adr/ADR-049-verified-attraction-and-meal-slots.md
  - ../../adr/ADR-042-no-city-encyclopedia-in-source.md
  - ../../adr/ADR-074-skeleton-meals-variant-a-only.md
  - ../../agent-specs/agent-stories.md
---

# fill 时无LLM按规则排餐

**产品选用（2026-09-20）。** 故事 `agent-meal-116`（AC Ready，未实现）。骨架维持 [ADR-049](../../adr/ADR-049-verified-attraction-and-meal-slots.md) D3 餐档；fill 零 LLM 现搜；用供应商字段排序/闸。

[骨架LLM搜餐](./research_restaurant_pre_search.md) **不实施**。[ADR-074](../../adr/ADR-074-skeleton-meals-variant-a-only.md) 保持 Proposed。

## 合同常量

| 符号 | 值 | 适用范围 |
| --- | --- | --- |
| `MEAL_RATING_MIN` | **3.5** | 有 `rating` 的卡（Google 与 AMAP） |
| `GOOGLE_USER_RATINGS_MIN` | **20** | 仅当 Google 卡 **已有** `user_ratings_total`；缺评论数则只走 rating |
| 走廊 | 800m → 2km → 5km | 不变；圆心仍是景点（S8） |
| Google 类型排除 | `cafeteria`、`food_court` | Places Table A 闭集；看 `primaryType` 或 `types[]` |
| AMAP 类型排除 | **无**（本故事） | `050000` 分不出食堂则只靠评分；禁止中文店名表 |

## 算法

1. `search_restaurants` 走廊命中（及 spend 过滤）后，卡须有 `native_id` + 坐标 + 在当前环内。
2. Google：`cafeteria` / `food_court` 丢弃。AMAP：不按 type 丢。
3. **过闸：** 有 rating 则须 `>= 3.5`；Google 另若有 `user_ratings_total` 则须 `>= 20`。无评分：**不淘汰、不当冠军**（本环有过闸店则忽略无评分；本环无过闸店才把无评分带去外环）。
4. 过闸集合内按 rating 高优先，同分近优先。未用过：先 `native_id`，再名。
5. 扩环直至 5km。5km 仍无过闸店：**仍选全集合最高分并落店**（禁 `meal_skipped`），`stop_display.notes` 含协议 id `meal_low_signal`（非用户文案，2play 默认不展示）。

## 禁止

店名/子串（内部、食堂、职工、管理处）、城市必吃、fill 使用 `CHAIN_DINING_DENY`、扩 `LANDMARK_AS_MEAL_DENY` 打食堂。

## 适配器（实现时）

Google search/Details fieldMask 增 `userRatingCount`；`PlaceCard.types` + 已有 `user_ratings_total`。MCP/fixture 同源。高德 mapper **不**为食堂加 type 排除。
