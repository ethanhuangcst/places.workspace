# ADR-006: Provenance-tagged results and client nav choice

## Status

**Superseded** by [ADR-052](./ADR-052-map-provider-routing.md) D6 (2026-09-06). Provenance + 客户端选 deeplink 仍有效，并入地图路由总则。

## Decision

Results carry `provider` / `sources[]` with logos and deep links; optional merge into clustered cards. The UI chooses which deep link to open based on **client** environment (e.g. mainland prefers AMAP when present).

## Consequences

Agent stays a data gateway; presentation and nav preference stay in apps.
