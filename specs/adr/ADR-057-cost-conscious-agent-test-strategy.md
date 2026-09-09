# ADR-057: Cost-conscious agent test strategy (fixture CI + probe cache + stops pool feed)

Family: places-agent testing / true-agent POC.

## Status

**Accepted**（2026-09-07）

## Context

Google Maps moved to pay-per-call after quota exhaustion. True-agent work (`plan_trip`, full-loop probes) and live verification still need real vendor data, but default PR CI and repeated local probes must not burn budget.

Separately, the city **AttractionPoi** registry (ADR-049 / ADR-056) now holds durable verified attractions with resolved `photos[0]`. Empty English Lisbon pool blocked using the registry as offline feed; seed scripts rebuilt Lisbon / Hong Kong / Taipei pools to ≥100 photo-bearing rows.

The prior strategy ([agent-test-plan.md](../agent-specs/agent-test-plan.md) + [ADR-021](./ADR-021-live-vendor-no-fixture.md)) already required fixture-default CI and fail-closed live. It did not define:

1. How to re-run live probes without repeating Google search/media calls.
2. A hard daily call budget for probe scripts.
3. How seeded stops pools fit the test pyramid (vs city encyclopedia fixtures — forbidden by ADR-042).

## Decision

### D1 — Three cost layers (always apply)

| Layer | Mechanism | Cost |
| --- | --- | --- |
| **L0 Default CI** | `PLACES_VENDOR_MODE=fixture`, no map/LLM keys; Postgres service for Trip tables; Vitest uses **memory** PoiRegistryStore when `VITEST` is set | Zero vendor $ |
| **L1 Probe file cache** | Opt-in `PLACES_PROBE_CACHE_DIR` (e.g. `tmp/.probe-cache`); `searchPlaces` / `geocode` persist + reuse responses (24h TTL) | Re-runs ≈ free after warm |
| **L2 Probe budget gate** | Probe HTTP via `budgetFetch`; `GOOGLE_DAILY_BUDGET_CALLS` (default 200); abort when exceeded | Hard daily ceiling |

ADR-021 honesty still holds: live mode must not silently serve fixtures. L1 caches **live** responses, not fixture cards.

### D2 — Stops pool is live feed, not a city encyclopedia

- Seed with destination-agnostic keyword templates (`${city} museum`, …) via [`scripts/seed-city-pois.ts`](../../places-agent/scripts/seed-city-pois.ts). Prefer https photo-bearing cards. Target ≥100 per city for skeleton density (`CANDIDATE_CAP` / 3-day minAttr).
- Seeded cities (as of 2026-09-07): Lisbon, Hong Kong, Taipei (Google/dual)；杭州、西安、上海、厦门（AMAP-only 文本分页 seed）。目标各 ≥100，photo% ≥90%，`must_see` 不入库（ADR-056）。
- Registry identity and upsert semantics: [ADR-056](./ADR-056-registry-backfill-semantics.md).
- **Forbidden:** growing per-city POI name lists in TypeScript (`discover-must-see` CATALOG / ADR-042).

### D3 — Where each layer sits in the pyramid

| Pyramid | Feed / store | Notes |
| --- | --- | --- |
| Unit (~70%) | Memory registry + injected search/geocode | `resetPoiRegistryStoreForTests` / `setPoiRegistryStore` per suite to avoid cross-test pollution |
| Integration (~20%) | Test Postgres Trip + optional Prisma registry | Future: SQL export of seeded pools for CI Prisma merge/diff-skip tests (not required for L0 green) |
| Live probe / UAT | Seeded AttractionPoi + L1 cache + L2 budget | `plan_trip` commit backfills pool (ADR-056); seed script bypasses `MUST_SEE_LIMIT` |

### D4 — CI wiring

Workspace workflow [`.github/workflows/places-agent-tests.yml`](../../.github/workflows/places-agent-tests.yml): fixture Vitest + Postgres on PR/push. No Google/AMAP/OpenAI keys in that job.

## Consequences

- Default PR stays zero vendor cost; live probes are opt-in and budget-capped.
- Repeated Lisbon/HK/Taipei probes after cache warm do not re-hit Google for identical search/geocode keys; registry upserts diff-skip when cardSlim unchanged (ADR-056).
- Operators must run seed / probe with real keys when refreshing pools; stale CDN photos are not auto-backfilled (ADR-051 D5).
- Destination `lookupKey` still embeds normalized city string — EN vs CN names fragment pools (known gap; not fixed in this ADR).

## Alternatives considered

| Alternative | Rejected because |
| --- | --- |
| Fixture-only forever (no live probes) | Violates ADR-021 live DoD; cannot accept true-agent quality |
| Replay only from handwritten JSON city packs in source | ADR-042 city encyclopedia risk |
| No budget gate (rely on cache alone) | Cache miss or cold start can still spike spend |
| Use `plan_trip` alone to grow pools | Commit upserts only `MUST_SEE_LIMIT` (5) chips — cannot reach 100 |

## Related

- [ADR-021](./ADR-021-live-vendor-no-fixture.md) — live honesty
- [ADR-042](./ADR-042-no-city-encyclopedia-in-source.md) — no city POI encyclopedia
- [ADR-049](./ADR-049-verified-attraction-and-meal-slots.md) — AttractionPoi scope
- [ADR-051](./ADR-051-discover-resolve-display-photo.md) — display photos
- [ADR-056](./ADR-056-registry-backfill-semantics.md) — registry match / diff-skip / no must_see
- [agent-test-plan.md](../agent-specs/agent-test-plan.md) §1.2 / §9

## Date

2026-09-07
