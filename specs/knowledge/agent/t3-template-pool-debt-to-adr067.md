# T3 template-pool quality debt → ADR-067

**as_of:** 2026-09-11  
**tags:** agent, itinerary, discovery, t3  
**related:**
- ADR-067
- agent-discover-110a
- agent-itinerary-103
- agent-itinerary-104

## Summary

MVP-T3 / T3+ shipped on the **code-template stops-pool** path (`skeletonPoolQueries` → pool → LLM pick). Usable Confirmed 2026-09-11. Known quality gaps were diagnosed and **intentionally not fixed on the as-built path**; they transfer to MVP-T3++ under [ADR-067](../../adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md):

| Gap | As-built cause | Hand-off |
| --- | --- | --- |
| Seasonal noise (e.g. 断桥残雪 in Sept) | Templates ignore season | LLM OptA nominate + hard season at discovery |
| Far-cluster day drift (3→5 days) | `ensureFarClustersOwnDays` silent peel | Validate-don't-repair + `deviations` |
| Thin POI destinations (北大壶 hang) | Vendor coverage thin; retry hang | Deviation + expand-radius with user confirm |

## Do not

- Grow city encyclopedias to paper over thin POI (ADR-042).
- Re-open T3+ stories to patch template pool once T3++ `110a` is in flight — implement ADR-067 instead.

## First implementation unit

`agent-discover-110a` (OptA nominate → clean → ground → delete template search).
