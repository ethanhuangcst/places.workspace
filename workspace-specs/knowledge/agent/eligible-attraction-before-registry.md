---
title: 可规划景点门槛先于目的地库
type: design-direction
status: active
as_of: 2026-09-04
tags:
  - discover
  - make_itinerary
  - adr-049
related_spec: 1.places-agent/agent-specs/0.refactor-plan.md
related:
  - adr/ADR-049-verified-attraction-and-meal-slots.md
  - adr/ADR-042-no-city-encyclopedia-in-source.md
---

# 可规划景点门槛先于目的地库

## Summary

合称（西湖十景、…名胜区）进芯片 / `must_include` 子串覆盖会导致 make 502。先做共用 eligible 谓词与餐档骨架；运行时景点库最后做，且不违反 ADR-042。

## Evidence

- 杭州 trip 池含「西湖十景」「杭州西湖风景名胜区」；骨架空、`POST /v1/make_itinerary` 502。
- F84 落地后未重启 `tsx server.ts` 时，UI 仍走旧校验（32s 502）。
- 重启后失败改为 `stop "白堤" not found`：LLM 写了池外景点；丢掉该站后杭州 3 日可用。
- ADR-042 禁的是源码城表，不是 PG 按 `native_id` 登记。

## Lesson / guidance

- Discover / make / 入库共用同一 filter，禁止三套「什么算真点」。
- `must_include` 对不上 eligible 点 → 降级，不整单失败。
- 缺 details ≠ 脏。漏网合称才用内部 `patchTrip` 从本 trip `candidates` 删除；不升 HTTP。
- 不要用「先建库」挡 S1。
- 改 `src/core` 后必须重启 agent（tsx 不热加载工具循环）。
- 池外景点：丢掉该站，不要为「白堤」类发明名整单失败。

## Links

- [ADR-049](../../adr/ADR-049-verified-attraction-and-meal-slots.md)
