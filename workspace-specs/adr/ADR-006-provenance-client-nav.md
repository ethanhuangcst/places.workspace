# ADR-006: Provenance-tagged results and client nav choice

## Status

Accepted

## Decision

Results carry `provider` / `sources[]` with logos and deep links; optional merge into clustered cards. The UI chooses which deep link to open based on **client** environment (e.g. mainland prefers AMAP when present).

## Consequences

Agent stays a data gateway; presentation and nav preference stay in apps.
