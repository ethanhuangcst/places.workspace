---
title: Fixture CI vs live vendors — honesty gate
type: ops-lesson
status: active
as_of: 2026-08-19
tags:
  - testing
  - maps
  - amap
  - tripadvisor
  - open-meteo
related_spec: 1.places-agent/agent-specs/4.test-strategy.md
related:
  - adr/ADR-021-live-vendor-no-fixture.md
  - adr/ADR-019-http-first-user-test-automation.md
  - knowledge/maps/vendor-adapters.md
  - knowledge/web-app-development/lessons-from-places-agent-mvp1.md
---

# Fixture CI vs live vendors — honesty gate

## Summary

Default CI on fixture vendors is correct. It is **not** proof that production vendors work. places-agent shipped AMAP search, Tripadvisor enrich, and Open-Meteo weather as “done” while live mode still served canned data. An operator found it with a real ChatBox query before production deploy. Binding decision: [ADR-021](../../adr/ADR-021-live-vendor-no-fixture.md).

## Evidence

- `getAdapter("AMAP")` ignored `PLACES_VENDOR_MODE` and always returned `amapFixtureAdapter` until 2026-08-19. `AMAP_API_KEY` only populated `configuredProviders()`.
- ChatBox `search_restaurants` with `providers: ["AMAP"]` returned `fixture_amap_*` (爱琴海 / 中环) for 紫藤路站烧烤. After AMAP live: real POIs (大茗烧烤, 二雷吉林烧烤, …).
- `src/adapters/open-meteo/` live forecast shipped 2026-08-19 (`GET /forecast`). Fixture matcher only when mode is not `live`. Opt-in: `make verify-open-meteo-live` (reuse healthy `PORT`; do not start a second Next in the same dir). Live + missing client still throws `open_meteo_live_unconfigured`. WMO `weather_code` 80 is real rain showers; the fixture signature is the triple `80` + 24/18 °C, not the code alone.
- Tripadvisor Terra live enrich shipped 2026-08-19 (`GET /locations/nearby`, name match). Fixture matcher only when mode is not `live`. Opt-in: `make verify-tripadvisor-live`.
- Tripadvisor Terra key is valid: `GET /locations/nearby` against `terra.tripadvisor.com` returned HTTP 200. Before this story the process never used it.
- Gates that stayed green: Vitest TC-H03/H05 (fixture names and canned ratings), `tools.test.ts` Shanghai AMAP fixtures, no `fixture_` ban in live mode, ChatBox TC-C deferred (ADR-019), MVP-2 AC status “implemented”, DoD never actually closed (typecheck scripts, E2E port, no coverage, no operator usability confirm).
- `common-test-strategy` allows fixtures in CI and forbids shipping mock-only features. Project tests equated fixture success with vendor DoD. Strategy file status was still **draft**.

## Lesson / guidance

| Allowed | Not allowed |
| --- | --- |
| Fixture adapters when `PLACES_VENDOR_MODE` is not `live` | Fixture cards, ratings, or forecasts when mode is `live` |
| Injected `fetchFn` in default CI | Claiming a vendor done with only fixture assertions |
| Opt-in `make verify-*-live` with real keys | Env key present ⇒ vendor is live |
| Skip / unconfigured / weather_unavailable if live client missing | Silent fallback to fixture so the demo looks full |

**Honesty checks (must exist per live vendor):**

1. Live mode + injected or real client → `sources[].native_id` does not start with `fixture_`.
2. Live mode + missing live client → skip/omit, **zero** fixture payloads.
3. Opt-in script hits the real host; fails the build/job if `fixture_` appears.

**DoD:** binding checklist and honesty matrix live in [4.test-strategy.md](../../../1.places-agent/agent-specs/4.test-strategy.md) **§1.1 Vendor live DoD**. A vendor story is not done until those items pass and the operator has seen a real-pin result (not only `make test`). AC status must be `live-honest` / `fail-closed` / `fixture-only` — not `implemented`.

**Do not** treat ChatBox as the only live detector. HTTP opt-in probes must run before production.

## Links

- [ADR-021](../../adr/ADR-021-live-vendor-no-fixture.md)
- [4.test-strategy.md](../../../1.places-agent/agent-specs/4.test-strategy.md) §1.1
- [vendor-adapters.md](../maps/vendor-adapters.md)
