# ADR-075: Tips during fill via BFF artifact poll

Family backlog: [`product-backlog.md`](../product-backlog.md) · story `2play-plan-90e`

## Status

**Accepted**（2026-09-21）— 产品确认 fill 进行中贴士可用。

**Related:** [ADR-046](./ADR-046-trip-store-pg-memory-fetch.md)（artifacts 只经 fetch）、[ADR-063](./ADR-063-skeleton-only-plan-trip.md)（`skeleton_only` 与 fill 分次）。

## Context

`skeleton_only` 在后台 dualWrite `artifacts.tips`。Fill 是另一次 `POST /api/plan` NDJSON，agent 写不进这条已经打开的流。只在 fill 开始和结束各 fetch 一次时，贴士要等整段 fill 结束才出现。

规格曾写成「agent dualWrite 后推 NDJSON」。那条推送没有可落地的 socket。

## Decision

### D1 — BFF polls during fill

`planItinerarySkeletonFill` 在首包、每个 `stop_filled` 之后、以及循环结束后，各 `fetch_trip_details` `fields: ["artifacts"]` 一次。载荷相对上次有变化时 yield `{ type: "tips" }`。每站最多一次，避免空转。

### D2 — Visa does not block the first tips read

`visa_requirement` 在第一次 artifacts 读取之后启动，不 `await` 它才读 tips。签证写入完成后，后续 poll 经 `travelTipsPayloadFromSlice` 并入同一 `tips` 事件。

### D3 — Panel mounts with the skeleton

T3 骨架上屏、fill 等待开始之前，plan 页把贴士区设为 loading。Fill 开始时再次置 loading 无害。

### D4 — Not a return-visit hydrate

本决定不覆盖离开 Plan 再回来。那是 `2play-plan-106`：`GET /api/plan/current` 必须另 fetch `artifacts`。

## Rationale

在 fill 流里轮询 store，和「写完立刻出现」的产品结果一致，又不给 agent 增加第二条到浏览器的通道。真推送需要 agent 持有 fill 的响应流，当前架构没有这条连接。

## Consequences

- 贴士最早出现在某一站 fill 完成之后的 poll，不是 dualWrite 的同一毫秒。
- Visa 与 `plan_next_stop` 可能交错改 revision；poll 会把 fetch 到的 revision 写回 fill 循环。
- `2play-plan-106` 仍须单独做。不要把政策正文写入 `itineraryJson`。

## Date

2026-09-21
