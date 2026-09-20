---
title: agent-meal-117 development plan
type: design-direction
status: confirmed
as_of: 2026-09-20
tags:
  - fill
  - google
  - meals
related:
  - ../../agent-specs/agent-design.md
  - ../maps/google-restaurant-search-latency.md
---

# `agent-meal-117` 开发计划（已确认）

**设计合同：** [agent-design §4.1](../../agent-specs/agent-design.md#meal-117-search)（A+C+B，不新开 ADR）。故事 GWT：`agent-stories.md` `agent-meal-117`。

**确认（2026-09-20）：** (1) §4.1 为合同；(2) 步骤 1–5 接受现稿，只走复核 + DoD usable（步骤 6）。探针不重跑。

## 范围

| 做 | 不做 |
| --- | --- |
| A 搜次封顶；C 搜餐超时不 MCP；B 泛餐饮 Nearby | 改 Directions；骨架 LLM 搜餐；城市餐厅表；2play 文案；高德 around |
| vitest TC-M117-01..07 | 把 Lisbon/台北探针当 CI 红线 |
| 探针 Lisbon + 台北对照 fill_s | 新 ADR |

## 步骤

| # | 内容 | 状态 |
| --- | --- | --- |
| 1 | **A** `resolveMealVenue`：过闸即停；空才下一点；无过闸才 cafe | **Done**（TC-M117-01..03） |
| 2 | **C** `searchRestaurants` timeout/abort 不调 Worker；502 仍 MCP | **Done**（TC-M117-04..05） |
| 3 | **B** `near`+泛餐饮 → `searchNearby`；菜名仍 `searchText` | **Done**（TC-M117-06..07） |
| 4 | 回归 meal-116 + 既有 `plan-next-stop` 餐测 | **Done**（2026-09-20 复核：`plan-next-stop` / `live` / `direct` 84 tests green） |
| 5 | 探针 `probe-t5-fill-review.ts lisbon taipei` | **Done**（fill_s 记录见 [`meal-117-lisbon-taipei-probe.md`](./meal-117-lisbon-taipei-probe.md)；不重跑） |
| 6 | **DoD usable confirm**；再 commit/push | **Pending**（待你确认可用） |

## 风险

- Nearby `locationRestriction.circle` 与 searchText 的 bias 行为不同；过闸靠 116 不靠「距离第一家」。
- Worker 可能无 Nearby：超时不再 MCP，真断网仍 searchText MCP。
- fill_s 含 Directions，探针不是纯搜餐秒数。
