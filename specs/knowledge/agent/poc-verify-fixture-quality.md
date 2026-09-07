---
title: POC fixture 验证必须模拟时钟链与餐店卡
type: ops-lesson
status: active
as_of: 2026-09-07
tags:
  - places-agent
  - plan_trip
  - testing
  - poc
related:
  - ../../adr/ADR-054-poc-before-ui.md
  - ../../adr/ADR-057-cost-conscious-agent-test-strategy.md
  - ../agent-specs/real-agent-refactory.md
---

# POC fixture 验证必须模拟时钟链与餐店卡

`verify-poc-true-agent.ts` 用 `_testMakeItinerary` / `_testPlanNextStopFill` 避免 Google 与 LLM。人眼看 HTML 时，**行程质量看起来像产品输出**。

固定 `09:00–11:00`、骨架只有上午、`meal.card=null` 会制造假阴性：评审误判真实 `planNextStopFill`（`prevEnd + leg`、餐窗 snap、`resolveMealVenue`）有缺陷。

验收脚本的 fill mock 至少要：

1. 用上一站 `end_time` 链式顺延，禁止同日时段重合
2. 每天 skeleton 含午餐 + 晚餐 + 至少一站下午
3. 匿名 meal 写入餐厅 `PlaceCard`

质量检查项：`no_time_overlap` / `has_afternoon` / `meal_has_card`。
