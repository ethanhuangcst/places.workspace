---
title: 故事状态审计 — as-built 与开放清单
type: ops-lesson
status: active
as_of: 2026-09-02
tags:
  - stories
  - dod
  - backlog
related_spec: 1.places-agent/agent-specs/agent-stories.md
related:
  - agent/trip-store-adr046-resolution.md
  - adr/ADR-046-trip-store-pg-memory-fetch.md
---

# 故事状态审计 — as-built 与开放清单

## Summary

在 Trip Store 规格化时，对 `agent-stories.md` 做了一次 as-built 对照：多条长期 ToDo 实际已在代码落地（visa、iconic、travel_tips、MCP stateless、骨架确定性修复）。将已落地标 Done、作废项标 Cancelled，开放项写入可发现清单，避免遗漏与重复立项。

## Evidence

| Feature | 曾标 | 审计后 | 依据 |
| --- | --- | --- | --- |
| 40 | ToDo | Cancelled | arrange 删除后无适用工具 |
| 48–52 | ToDo | Done | `orizn/`、`find-iconic-places`、`travel-tips`、别名、`stateless /mcp` |
| 62 | ToDo | Done | `reseatStayToDayOrigin` / `dropCityNameStops` + TC-M15 |
| 39 / 41 | ToDo | ToDo | tsc / opt-in 仍欠 |
| 45 | 部分 | 部分 | arrange 硬删仍 gate 2play |
| 63–66 | — | ToDo | ADR-046 未实现 |

开放清单落点：`agent-specs/e2e-test-result/04-rome.md`「开发计划」+ `0.refactor-plan.md` 遗留评估 + where2play 任务表。

## Lesson / guidance

1. 大架构切片开工前，先跑 **故事 × 代码** 审计，更新总表与状态行。
2. Cancelled 保留短记录，勿从历史抹掉。
3. 跨仓依赖（agent F65 ↔ 2play plan-46）必须同时改两边 stories，并指向同一 refactor 批次。
4. e2e 结果文件若挂人工开发计划，注明「勿被脚本覆盖」。

## Links

- `agent-stories.md` 总表变更摘要（2026-09-02）
- `0.refactor-plan.md`「遗留功能清单评估」
