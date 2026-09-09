# ADR-055: MVP 重切为真智能体闭环

Family backlog: [`product-backlog.md`](../product-backlog.md)

## Status

**Accepted**（2026-09-06）

## Context

[ADR-050](./ADR-050-where2play-no-product-llm.md) Accepted 后，where2play 行程编排迁到 places-agent 真智能体环。as-built 的 MVP-24（`2play-plan-37` Lisbon 4D usable 等）在 `discover_places` → `make_itinerary` → `plan_next_stop` 管线上打磨，存在质量问题不可验收，且其 BFF 编排、chat 路由、replan 逻辑在 Target 落地时大部分要重写或丢弃。

继续在 as-built 上打磨再做 true-agent = 明显返工。直接跳到 true-agent = 中间无可演示产品。

## Decision

### D1 — MVP-24 暂停

MVP-24 as-built 打磨行状态改 `Paused`（见 [`../product-backlog.md`](../product-backlog.md) §1 Legend），待真智能体主干稳定后再排期。不删行，保留追溯。

### D2 — MVP 重切为真智能体闭环

每个 MVP 批次 = 一个真智能体能力最小闭环 + 2play 消费该闭环。agent 与 2play 保留各自功能编号（producer/consumer 边界，ADR-039），同属一个 MVP 批次。

批次序（详见 [`../product-backlog.md`](../product-backlog.md) §0 / §1 `true-agent` / `TA` 行）：

| 批次 | agent 能力 | 2play 消费 |
| --- | --- | --- |
| POC | intake + 必去芯片 + fetch candidates | 无（脚本） |
| MVP-T1 | intake + need_input + 芯片 | 5 题问卷 + 第 6 题芯片勾选 |
| MVP-T2 | 补池 + 骨架 + 按日 filled（无餐） | 行程详情逐日渲染 |
| MVP-T3 | 餐档 + directions + 硬闸 | 行程详情含餐与交通 |
| MVP-T4 | 四卡（artifacts） | 出行贴士页 |
| MVP-T5 | chat 改行程 | in-page chat |
| 扩展 | 杭州/香港/起点卡探针 | 三城 + 起点卡消费 |

### D3 — AC 顺序

POC 的 AC 先写（取细化检查表 #1/#5/#8/#25/#28）。MVP-T1+ 的 AC 在 POC 通过后再写，避免在未验引擎上固化契约。

### D4 — what2eat 隔离不变

MVP-T 批次不得把 `search_restaurants` / 2eat `chat` / `geocode` / `get_place_details` 并入 `plan_trip`（[ADR-050](./ADR-050-where2play-no-product-llm.md) D3）。每批显式列「不改 2eat」。

### D5 — 唯一 backlog 原则保持

所有真智能体行与 2play 消费行均在 [`../product-backlog.md`](../product-backlog.md) §1。stories 文件只放 GWT/AC，不放排期与 high-level requirement。

## Consequences

- 暂停期无 as-built 可演示产品；POC 可观测物（[ADR-054](./ADR-054-poc-before-ui.md) D4）替代可见进展。
- 避免在 as-built 管线上返工。
- producer/consumer 边界清晰，ADR-039 验收不混。
- MVP-T 批次依赖 POC 通过；POC 拖长会阻塞后续。

## Related

- [ADR-050](./ADR-050-where2play-no-product-llm.md) Accepted
- [ADR-054](./ADR-054-poc-before-ui.md) POC 先于 UI
- [ADR-039](./ADR-039-cross-product-as-built-vs-target.md) as-built vs Target
- [`../agent-specs/agent-design.md`](../agent-specs/agent-design.md) 真智能体能力清单
