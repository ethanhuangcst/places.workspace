# ADR-022: Timed itinerary contract (`detail: timed`)

## Status

Accepted (amended 2026-08-19)

## Context

MVP-2 `plan_itinerary` only redistributed caller `places[]` into day buckets. Callers need an hour-by-hour day engine (origin hotel, visit slots, meals, multi-mode legs, weather buffers) without inventing POIs or ETAs.

## Decision

1. Keep one public tool `plan_itinerary` (HTTP + MCP). Add `detail: "stops" | "timed"` (default `stops` for backward compatibility).
2. `detail: "timed"`: `origin` optional. `days[].day_index` is 1-based. When origin is omitted, `search_anchor` is the **city** parsed from `natural_language` / destination name (not a tower/station pin). Empty `places` → server-side `search_places` with **attraction allow / plaza-mall-station-scenic deny** (never unfiltered lodging). Always emit every calendar day in `bounds`. Optional `destination` is echoed and biases later-day place buckets; last visit of last day may include `legs_to_destination`.
3. Weather (Open-Meteo only) drives deterministic `planning_impact` and walk `weather_buffer_min`; never LLM-invented weather or ETAs.
4. Meals are `blocks` with `kind: "meal"` (`lunch` | `cafe` | `dinner`) from live `search_restaurants` (dining allow/deny; landmark names are not restaurants even if `category=restaurant`). **Lunch** is the gap between morning and afternoon visits. **Dinner is 18:00–20:00**. If the last visit ends before 17:00, an optional cafe/tea block fills until 18:00. Identity key = lowercase `native_id` or normalized name. Every visit and every meal **option** is unique for the whole trip (same day and across days); visit vs meal are mutually exclusive. After 1–2 extra searches, omit the meal/visit rather than reuse a venue. Legs with `duration_min > 300` are dropped. When `PlaceCard.hours` is mapped, options that do not overlap the meal window are dropped; missing hours stay as `hours_unknown`.
5. Legs prefer **configured `providers[]` Directions**. Locale CN/HK/TW **and** AMAP already in `providers[]`: timed search tries AMAP first, then Google fill. Never inject AMAP if the caller omitted it (ADR-005). AMAP search retries once on `provider_failed`. Failures do not invent durations; `skipped` names the failing provider id(s).
6. `PlaceCard.hours` is populated only when Google `regularOpeningHours` or AMAP `opentime_*` is present; Tripadvisor leaves hours unset when the API has none.

## Consequences

- where2play / ChatBox render JSON (including optional markdown later); agent stays the engine ([ADR-008](./ADR-008-itinerary-ownership.md)).
- More vendor calls per plan (search + restaurants + directions) → latency and quota; cap options and concurrency as needed.
- Local `.env` with `GOOGLE_DIRECT_FORCE_FAIL=1` skips Google Directions; production must not set that flag. AMAP Directions uses `AMAP_API_KEY` independently.
- Callers still choose `providers[]` (ADR-005); geo does not rewrite the list.

## Date

2026-08-19
