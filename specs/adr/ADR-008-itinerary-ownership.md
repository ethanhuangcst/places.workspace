# ADR-008: Itinerary engine vs trip UX ownership

Family backlog: [`product-backlog.md`](../product-backlog.md)

## Status

Accepted

## Decision

`plan_itinerary` lives in places-agent. where2play owns trip presentation and workflow.

## Consequences

Reuse planning for other callers; where2play stays thin.
