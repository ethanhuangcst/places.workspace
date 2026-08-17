# ADR-005: Caller-driven multi-provider gateway

## Status

Accepted

## Decision

Callers pass `providers[]` (and enrich/merge options). places-agent does not embed a hard rule that mainland destinations must use AMAP only. This supersedes destination-bucket routing in `geo-capability-route.json` as a hard router.

## Consequences

Apps encode product policy; agent validates credentials/capabilities and tags results.
