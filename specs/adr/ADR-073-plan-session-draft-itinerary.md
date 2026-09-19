# ADR-073: Plan session draft itinerary (`PlanSessionCache` vs `SavedItinerary`)

Family backlog: [`product-backlog.md`](../product-backlog.md) · story `2play-plan-105`

## Status

**Accepted**（2026-09-19）— 产品确认：fill 细节进库为临时稿；显式保存走已保存行程。

**Related:** [ADR-046](./ADR-046-trip-store-pg-memory-fetch.md)（Trip Store 写/读）、[ADR-028](./ADR-028-decision-history-on-save.md)（保存快照）、[ADR-033](./ADR-033-where2play-postgres-prisma.md)（Postgres 模型）。

**Does not supersede ADR-046.** Agent 仍是 discover/make/fill 写路径真源；本 ADR 只定义 **where2play 产品 DTO** 在 App DB 的草稿与 hydrate 规则。

## Context

T3 fill 通过 `POST /api/plan` NDJSON 在 BFF 侧 upsert `PlanSessionCache.itineraryJson` 为完整 `ItineraryDto`。用户离开 Plan 页（例如打开「我的行程」）后 React 状态丢失。返回 Plan 时 `GET /api/plan/current` 调用 `refreshItineraryFromTripLedger`，用 `itineraryFromSkeletonFetch` 从 Trip Store **skeleton** 重建 days，常把已填 slot 冲成空 day（名称/餐站与 skeleton  attractions 不匹配）。`fetch_trip_details` 的 `filled` 字段是**最近一站**切片，不能拼成多日主区。

用户期望：未点「保存」的行程是**临时稿**，导航离开再回来应仍在；点「保存」才进 `SavedItinerary`；下一次规划或确认 Replan **覆盖**临时稿，不删已保存卡。

## Decision

### D1 — Draft board SoT

当 `PlanSessionCache.itineraryJson` 含至少一个 `kind === "place"` 的 slot 时，Plan 主区 hydrate **以该 JSON 为准**，不降级为 skeleton-only 空 days。

### D2 — Saved is explicit

`SavedItinerary` 仅由用户 `POST /api/saved` 创建。浏览「我的行程」**不**删除或替换 `PlanSessionCache`。

### D3 — Current route refresh

`GET /api/plan/current` 在 `criteria.tripId` 存在时仍可 `fetch_trip_details` 取 `skeleton`（助手 thread）并合并 `trip_id`/`revision`。**禁止**用 skeleton merge 覆盖 D1 中已填的 `itineraryJson`。

### D4 — Write path unchanged

Fill 进度仍由 `POST /api/plan`（及既有 NDJSON persist）写入 cache。Trip Store 仍由 agent 写工具更新；产品 DTO 是展示快照，因 `filled` 非全日 board。

### D5 — Overwrite rules

新 takeoff / `POST /api/plan/trip` / 确认 Replan（含 `DELETE /api/plan/current`）覆盖或清空临时稿。已保存行程独立。

## Consequences

- 实现：`refreshItineraryFromTripLedger` 在 cached 已填时保留 itinerary，仅附加 `skeleton`。
- 测试：BFF current + hydrate 单测；杭州 AMAP e2e 往返导航。
- 不新增表；不把 agent `filled` 当作 Plan 板唯一 rebuild 源。
