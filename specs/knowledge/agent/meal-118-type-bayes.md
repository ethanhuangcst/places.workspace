---
title: agent-meal-118 type gate + Bayesian rank
type: design-direction
status: done
as_of: 2026-09-20
tags:
  - fill
  - meals
  - google
related:
  - ./research_fill_rule_meals.md
  - ../../agent-specs/agent-design.md
---

# `agent-meal-118` — 正餐类型 + 贝叶斯排序

**产品拍板（2026-09-20）：** 方案 1 类型闸、2 贝叶斯 m=50 C=4.0、3 单独立项。**Done**（usable Confirmed 2026-09-20）。

## 证据（Lisbon D1 晚餐）

ARTIS CHUNXI · `ChIJZczcuvbLHg0RGXWAGxiyjKk` · R. dos Jerónimos 22A。Google types：`breakfast_restaurant`, `cafe`, `restaurant`, …；rating 5.0 / 46 评。官网 [chunxiart.pt](https://www.chunxiart.pt/) 为艺术工坊+咖啡，11:00–19:00。fill 因 Nearby `includedTypes=restaurant` + 裸 rating 排序 + 过闸即停选中。

## 合同

见 [agent-design §4.2](../../agent-specs/agent-design.md#meal-118-rank)。`pickMealVenue({ query })` + Nearby honest `primaryType`。

## HK lunch: 7 Paintings（2026-09-20）

`7 Paintings - Hongkong` Google `primaryType=restaurant`、`types` 含 `restaurant`/`food`，rating 4.4 / 219，`PRICE_LEVEL_VERY_EXPENSIVE`。现场是 Murray 酒店 immersive 七道菜晚宴秀，不是画廊。118 类型闸**正确放行**。午餐不合适来自默认 spend=2 不滤 `$$$$`、无营业时间闸（本故事明确不做）、套餐时长，不是类型错绑。不要用店名禁 Paintings。
