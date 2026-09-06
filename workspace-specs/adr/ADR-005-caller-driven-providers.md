# ADR-005: Caller-driven multi-provider gateway

## Status

**Superseded** by [ADR-052](./ADR-052-map-provider-routing.md) (2026-09-06). 显式 `providers[]` 覆盖仍成立；大陆自动 AMAP 与检测顺序以 052 为准。

## Decision

Callers pass `providers[]` (and enrich/merge options). places-agent does not embed a hard rule that mainland destinations must use AMAP only. This supersedes destination-bucket routing in `geo-capability-route.json` as a hard router.

## Consequences

Apps encode product policy; agent validates credentials/capabilities and tags results.
