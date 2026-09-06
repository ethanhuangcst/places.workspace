# ADR-054: POC 先于 UI（真智能体引擎以脚本验，再接 2play）

Family backlog: [`product-backlog.md`](../product-backlog.md)

## Status

**Accepted**（2026-09-06）

## Context

[ADR-050](./ADR-050-where2play-no-product-llm.md) 把 where2play 行程编排迁到 places-agent 真智能体环（`plan_trip` + `fetch_trip_details`）。as-built 的 `discover_places` → `make_itinerary` → `plan_next_stop` 管线将被替换。

风险：若直接在 2play UI 上接 Target 引擎，引擎闭环（工具环、事实闸、芯片验真、取图、供应商路由）一旦有问题，UI 与引擎耦合，定位慢，且 2play 消费侧返工。

[ADR-055](./ADR-055-mvp-reslice-true-agent-loops.md) 把 MVP 重切为真智能体闭环。第一个闭环需要一个不依赖 UI 的验证手段。

## Decision

### D1 — POC 先于 UI

真智能体引擎先用**脚本 / CLI** 验证最小闭环，不接 2play UI。POC 通过后再写消费 UI 的 MVP-T 批次。

### D2 — POC 范围（Lisbon 单城最小闭环）

`agent-poc-01`（见 [`../agent-specs/agent-stories.md`](../agent-specs/agent-stories.md)）：

- `plan_trip` 收 `city=Lisbon` → `geocode` 锚点 → 懒建 `trip_id` → LLM 提名必去 → `search_places`+eligible → `commit_trip` 写 `candidates`（`must_see` + `photos[0]`）
- `fetch_trip_details(fields:["candidates"])` 读验真芯片
- **不含**骨架、填站、餐、四卡、chat

### D3 — 通过判据取细化检查表

不写「符合设计」空话。通过判据 = AC1–AC4 全绿 + 细化检查表 #1 / #5 / #8 / #25 / #28 无违反（见 [`../knowledge/agent/real-agent-refinement-checklist.md`](../knowledge/agent/real-agent-refinement-checklist.md)）。

### D4 — POC 产出可观测物

trip JSON + 芯片渲染（HTML 或脚本 print），便于人眼复核。POC 期间无 UI 不等于无可见进展。

### D5 — 扩展探针在 POC 通过后

杭州（大陆 AMAP-only / 废除 discover 扩源 / D9-D10）、香港（Google+AMAP）、起点卡（[ADR-053](./ADR-053-origin-stay-as-stop-card.md)）是 POC 通过后的**扩展探针**，不并进 POC。

## Consequences

- 引擎问题在无 UI 噪声下暴露，定位快。
- 2play 消费侧返工面缩小到 POC 通过之后。
- POC 期间无可演示产品；用可观测物替代（D4）。
- POC 拖长会延长无 UI 窗口；POC 只做 Lisbon 单城最小闭环（D2）。

## Related

- [ADR-050](./ADR-050-where2play-no-product-llm.md) Accepted
- [ADR-055](./ADR-055-mvp-reslice-true-agent-loops.md) MVP 重切
- [`../agent-specs/real-agent-refactory.md`](../agent-specs/real-agent-refactory.md) 能力清单
