# ADR-021: Live vendor mode must not serve fixture data

## Status

Accepted

## Context

places-agent has two vendor paths: **fixture** (canned cards for default CI) and **live** (`PLACES_VENDOR_MODE=live`, production-shaped). Specs, user tests, and ChatBox treat `AMAP`, `GOOGLE_MAPS`, Tripadvisor enrich, and Open-Meteo as real capabilities. Env already held live keys.

Before 2026-08-19, `getAdapter("AMAP")` always returned `amapFixtureAdapter`. Tripadvisor enrich and Open-Meteo weather still only have `fixture.ts`. `configuredProviders()` in live mode advertised AMAP/Tripadvisor as configured whenever the API key was set. Default Vitest asserted fixture names, `fixture_*` ids, and canned ratings as the contract. `make test` stayed green. ChatBox MCP user tests were deferred (ADR-019). MVP-2 AC was marked implemented.

An operator discovered the gap with a real query (紫藤路站附近烧烤) against live mode: AMAP returned 爱琴海/中环 fixture restaurants. A later probe showed Tripadvisor Terra **keys work** (`GET /locations/nearby` HTTP 200) but the process never called Terra. Open-Meteo itinerary weather is the same class of lie (always `weather_code: 80`).

`common-test-strategy` already forbids shipping features that only work against mocks, and allows fixtures for **CI**. The hole: fixture adapters remained reachable in **live/production mode**, and fixture assertions were treated as DoD for vendor features.

## Decision

1. **`PLACES_VENDOR_MODE=live` is fail-closed.** A configured vendor must use a live client. If the live client does not exist, the vendor is **unavailable** (skip / omit / `errors.provider_unconfigured` or `errors.weather_unavailable`). It must **not** fall through to fixture cards, ratings, or forecasts.
2. **Fixture mode is test-only.** Default CI (`make test`) stays fixture. Fixture assertions must not be used to claim a live vendor is done.
3. **DoD for a vendor capability** requires all of:
   - Live client exists and is selected in live mode
   - Unit tests inject `fetchFn` (no live keys in default CI)
   - At least one opt-in probe with a real key (`make verify-amap-live`, `make test-live`, or equivalent) that asserts **no** `fixture_` native ids and provenance matches the vendor
4. **`native_id` prefix `fixture_` is forbidden** in live mode responses. CI must include an honesty test for each live vendor.
5. **Marking a story “implemented”** in AC is not DoD. DoD still requires the quality checklist, including this honesty gate, and operator confirmation for user-facing behavior.

## Rationale

Rejected: keep serving fixture in live mode until each live adapter is written (looks done; ships lies). Rejected: per-vendor env flags as the only control (`PLACES_VENDOR_MODE` already exists; the bug was not honoring it per adapter). Rejected: relying on ChatBox/manual tests as the live gate (ADR-019 deferred them).

Fail-closed is the only option that makes a missing live adapter **visible** before production.

## Consequences

- AMAP live client (2026-08-19) is the model: `getAdapter` branches on mode; injected-fetch tests; `verify-amap-live`.
- Tripadvisor enrich and Open-Meteo must skip in live mode until live clients exist. Production with `PLACES_VENDOR_MODE=live` must not attach canned Tripadvisor ratings or canned `weather_code: 80`.
- Google already had a live path; keep the opt-in Worker probe (`verify-gmaps-fallback`).
- Follow-up stories: Tripadvisor Terra live enrich; Open-Meteo live forecast. Do not mark Feature 8 or weather as done until those probes pass.
- Test strategy § vendor honesty is binding, not draft flavor text.

## Date

2026-08-19
