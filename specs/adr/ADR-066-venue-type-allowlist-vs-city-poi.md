# ADR-066 — Venue-type allowlists vs city→POI tables

- **Status:** Accepted
- **Date:** 2026-09-10
- **Related:** [ADR-042](./ADR-042-no-city-encyclopedia-in-source.md) · [ADR-065](./ADR-065-nominate-vs-skeleton-relationship.md)

## Context

T3 skeleton pool filters dropped theme-park-shaped search hits (e.g. English `*Resort*` matched lodging; CN `…乐园` / `娱乐场所` missed `ATTRACTION_ALLOW`). Fixing that without encoding city→POI rows (ADR-042) needs a clear line.

## Decision

1. **Allowed:** Destination-agnostic **venue-type** templates in allow/deny regexes and pool queries (`乐园` / `主题公园` / `theme park` / `amusement` / zoo / aquarium). Same class as existing `museum|park|景点`.
2. **Forbidden:** City→named POI maps or brand rows in production source (e.g. Shanghai→迪士尼). Hardcode guard + ADR-042 still apply.
3. **Lodging:** Bare `resort` must not reject visit venues that also carry attraction/amusement signals (category or name templates). Real hotels (`hotel` / 宾馆 / 酒店 / …) stay lodging.
4. **REAL AGENT:** Filters enable tool hits into the pool; the model still chooses; validator stays pool-only. Explicit must-see remains nominate (T4), not skeleton invention.

## Consequences

- Stories `agent-itinerary-105`–`107` implement allow + kids queries + soft prompt ranking.
- Production source must not add denylisted city landmark tokens; tests may use them as fixtures.
