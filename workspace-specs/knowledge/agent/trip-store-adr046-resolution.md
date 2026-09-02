---
title: Trip Store 决议过程与落地清单（ADR-046）
type: design-direction
status: active
as_of: 2026-09-02
tags:
  - trip-store
  - mcp
  - where2play
  - adr-046
related_spec: 1.places-agent/agent-specs/0.refactor-plan.md
related:
  - adr/ADR-046-trip-store-pg-memory-fetch.md
  - adr/ADR-045-iconic-places-unified-acquisition.md
  - adr/ADR-025-places-agent-postgres-prisma.md
  - agent/multi-agent-concurrent-editing.md
---

# Trip Store 决议过程与落地清单（ADR-046）

## Summary

2026-09-02 将原 TBD-1 收敛为 [ADR-046](../../adr/ADR-046-trip-store-pg-memory-fetch.md)：服务端权威行程（PostgreSQL + 内存热副本）、懒创建 `trip_id`、读经 `fetch_trip_details`、尽快删 `display_current_stop`、不新增 `start_trip`、`patch_skeleton` 仅内部。where2play plan-46 须与 agent MVP-16 **同窗**，禁止先接 display 再二次改造。开放故事清单写入 `e2e-test-result/04-rome.md` 开发计划以免遗漏。

## Evidence

- 产品目标双主：多工具改同一行程；宿主更好用行程数据/上下文（不只为抠 LLM 秒数）。
- 读路径否决宿主直连 DB；BFF 持库仍算服务端读。
- 存储：PG 权威 + 内存热副本 + `revision` 乐观锁（非真双主）。
- 工具精简：不新增 `start_trip`；懒创建；删 display；新增仅 `fetch_trip_details`。
- 议题 B（按日并发骨架）探针：平均不加速 → 默认不切。
- Orizn 签证走 places-agent **REST**（ADR-044），与 Cursor IDE 的 Orizn MCP 无关。
- 故事审计：F48–52 / F62 已 as-built Done；F40 Cancelled；开放 F39/41/45 剩余/63–66 + 2play 37–39。

## Lesson / guidance

1. **架构决议用逐项确认**，再写 ADR；TBD 文件决议后删除，避免双真源。
2. **存储默认对齐已有引擎**（ADR-025 Postgres）；「内存先行」仅当无库或纯试契约时考虑。
3. **宿主镜像分级**：可控 BFF 全量 hydrate；不可控 MCP 宿主弱镜像 + fetch。
4. **消费端与破坏性工具变更同窗**（plan-46 ↔ 删 display），避免中间态返工。
5. **开放 ToDo 挂到可发现锚点**（如成功 e2e 样例文件末尾「开发计划」），并交叉 refactor-plan / 2play-stories。
6. **as-built vs 故事状态**定期审计，避免已实现功能长期标 ToDo 导致重复立项。

## Links

- ADR-046、MVP-16：`1.places-agent/agent-specs/0.refactor-plan.md` 批次 16
- 开放清单：`1.places-agent/agent-specs/e2e-test-result/04-rome.md`（人工段）
- where2play：`3.where2play/2play-specs/2play-stories.md` Feature 37 AC8 / W2.5–W2.6
