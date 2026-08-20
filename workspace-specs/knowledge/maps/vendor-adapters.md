---
title: Map and weather vendor adapter notes
type: research-note
status: active
as_of: 2026-08-20
tags:
  - maps
  - amap
  - google
  - open-meteo
  - tripadvisor
  - mcp
related_spec: workspace-specs/3.tech-specs.md
related:
  - knowledge/handbook.md
  - knowledge/maps/places-capabilities.md
  - knowledge/i18n/hk-tw-output.md
  - knowledge/agent/places-agent-loop.md
  - adr/ADR-005-caller-driven-providers.md
  - adr/ADR-007-tripadvisor-match.md
  - adr/ADR-014-open-meteo-weather.md
  - adr/ADR-017-gmaps-mcp-fallback.md
  - adr/ADR-020-http-only-chat-and-enrich.md
  - adr/ADR-021-live-vendor-no-fixture.md
  - adr/ADR-024-quality-gates-typescript-7.md
---

# Map and weather vendor adapter notes

Reusable integration facts for places-agent adapters. **Decisions:** [ADR-005](../../adr/ADR-005-caller-driven-providers.md) (caller `providers[]`), [ADR-007](../../adr/ADR-007-tripadvisor-match.md) (Tripadvisor name+location), [ADR-014](../../adr/ADR-014-open-meteo-weather.md) (Open-Meteo only), [ADR-017](../../adr/ADR-017-gmaps-mcp-fallback.md) (Google direct then Worker MCP), [ADR-020](../../adr/ADR-020-http-only-chat-and-enrich.md) (chat and enrich HTTP-only), [ADR-021](../../adr/ADR-021-live-vendor-no-fixture.md) (live mode must not serve fixture). No secrets in this file.

## Summary

AMAP Web 服务 uses **lng,lat** and GCJ-02 in CN/HK/MO/TW. Google Places is WGS-84; when direct REST fails, use the Cloudflare Worker MCP and still tag `GOOGLE_MAPS`. Weather is Open-Meteo (WMO codes), not AMAP/Google weather. Open-Meteo has no language parameter — translate `weather.wmo.{code}`. Tripadvisor enrich is HTTP search only (`enrich.tripadvisor`), not a `providers[]` vendor and not an MCP tool.

## Evidence

- AMAP docs: [Web 服务概述](https://lbs.amap.com/api/webservice/summary), [地点搜索 2.0](https://lbs.amap.com/api/webservice/guide/api/newpoisearch), [地理编码](https://lbs.amap.com/api/webservice/guide/api/georegeo), [路径规划](https://lbs.amap.com/api/webservice/guide/api/direction/), [坐标系 FAQ](https://lbs.amap.com/faq/advisory/others/39840/).
- Google Weather coverage: [supported countries](https://developers.google.com/maps/documentation/weather/coverage) — CN unsupported; HK supported.
- Open-Meteo: [forecast docs](https://open-meteo.com/en/docs), [commercial vs free](https://open-meteo.com/en/pricing), [CC BY attribution](https://open-meteo.com/en/licence).

## Lesson / guidance

### AMAP (高德 Web 服务)

| Rule | Detail |
| --- | --- |
| Key type | **Web 服务 API** key. JS API keys / `securityJsCode` are not this adapter. |
| Host | `restapi.amap.com` (`AMAP_BASE_URL=https://restapi.amap.com`) |
| Coord order | `location` / `origin` / `destination` = **lng,lat** (longitude first), ≤6 decimals. Not Google `lat,lng`. |
| CRS | GCJ-02 (高德坐标系) in CN / HK / MO / TW; WGS-84 overseas. Convert GPS via `GET /v3/assistant/coordinate/convert` (`coordsys=gps`) before mainland search/route. |
| Dining | POI `types=050000` (餐饮服务) for `search_restaurants`. |
| Search | Prefer v5: `/v5/place/text`, `/v5/place/around`. Geocode: `/v3/geocode/geo`, `/v3/geocode/regeo`. Driving: `/v3` or `/v5/direction/driving`. |
| Not | Tripadvisor replacement. Do not call AMAP `weatherInfo` (ADR-014). |
| **Taiwan** | AMAP has **poor Taiwan coverage**. Provider resolver explicitly excludes Taiwan (台北/台中/台南/高雄 markers + coords lat 21.9–25.3 lng 120–122). Taiwan searches use Google only. |
| **Auto-select** | ADR-026: 大陆目的地 → AMAP only; 香港 → AMAP + Google; 其他 → Google only. Caller `providers[]` overrides. |
| Timed CN | Chinese `locale` plus caller-listed AMAP: timed `plan_itinerary` searches AMAP **first**, then Google fill. Chinese names on Google cards are not Gaode. |
| CRS pin | `near.crs === "GCJ-02"` → **skip** GPS convert; use lng,lat as-is. WGS / omitted crs → `coordsys=gps` convert. |
| Around radius | Dining around **1000 m**; non-dining places around **15000 m** (city POIs, not 1 km hotel bubble). |
| Assembled queries | Locale CN/HK/TW **or** CJK in origin/destination/NL → vendor `keywords` in Chinese (CN simplified, HK/TW traditional). **Never rewrite** the caller’s `query` field. |
| Corridor | Empty `places` + origin + dest + `numDays > 1` → search near interpolated pins along the corridor, not one city-center pool. |
| Trust | Key only on places-agent. Deeplinks secret-free. |
| Live search | `near` already GCJ-02 → around without convert. WGS `near` → convert `coordsys=gps` then `/v5/place/around`. `address` without `near` → `/v3/geocode/geo` then around. Dining `radius=1000` + `types=050000`; places around `radius=15000` without dining type. Keywords only → `/v5/place/text`. Cuisine maps into keywords (`barbecue` → `烧烤`). HTTP 200 with `status != 1` is failure. |
| Implemented | places-agent [`1.places-agent/src/adapters/amap/`](../1.places-agent/src/adapters/amap/) — `config.ts`, `direct.ts`, `card-mapper.ts`, `keywords.ts`, `live.ts`. Fixture remains when `PLACES_VENDOR_MODE` is not `live`. |

### Google Maps (direct + Worker MCP)

| Rule | Detail |
| --- | --- |
| Direct | `maps.googleapis.com` + `GOOGLE_MAPS_API_KEY` on places-agent. Places API (New), Geocoding, Routes. CRS WGS-84. |
| Fallback | On egress failure only: Streamable HTTP MCP `GMAPS_MCP_URL` + `GMAPS_MCP_BEARER`. `tools/list` first. This is **not** places-agent `/mcp`. |
| Provenance | Always `GOOGLE_MAPS`. Do not invent provider id `GMAPS_MCP`. |
| Keys | Worker holds the Maps key. Agent sends MCP bearer only. No `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY`. |
| Do not | Fall back to AMAP unless the caller asked for AMAP. Worker does not fix mainland **client** Google Maps app/web (ADR-006). |
| Dev test | `GOOGLE_DIRECT_FORCE_FAIL=1` simulates egress failure (VPN on); rejected when `NODE_ENV=production`. See `scripts/verify-gmaps-fallback.sh`. |
| Implemented | places-agent [`1.places-agent/src/adapters/google/`](../1.places-agent/src/adapters/google/) — `direct.ts`, `mcp-client.ts`, `live.ts` (ADR-017). |

### Open-Meteo (weather)

| Rule | Detail |
| --- | --- |
| Call | `GET /v1/forecast` with `latitude` + `longitude`. Not a `providers[]` vendor. |
| Free vs commercial | Free `api.open-meteo.com` is non-commercial. Production: `customer-api.open-meteo.com` + `OPEN_METEO_API_KEY`. Attribution required (CC BY). |
| Language | **No** `language` param. Body is numeric (`weather_code`, °C, %, m/s). Docs are English. |
| Display | Map `weather_code` → `weather.wmo.{code}` in `EN` / `CN` / `HK` / `TW`. Format numbers with `Intl`. LLM narrative uses localized strings — never English “Slight rain showers” in CN/HK/TW. |
| Live search | `GET /forecast?latitude=&longitude=&daily=weather_code,temperature_2m_max,temperature_2m_min&timezone=auto&forecast_days=16`. Optional `apikey` on customer host. HTTP fail → omit weather, keep itinerary (`errors.weather_unavailable`). |
| Implemented | places-agent [`1.places-agent/src/adapters/open-meteo/`](../../../1.places-agent/src/adapters/open-meteo/) — `config.ts`, `direct.ts`, `live.ts`. Fixture remains when `PLACES_VENDOR_MODE` is not `live`. |
| Timed itinerary | `detail:timed` builds clock slots, weather `planning_impact`, **visit-gap meal windows**, and legs. Auto-search uses attraction allow / lodging deny (no unfiltered fallback). Destination biases later days. Google Directions JSON and/or AMAP `/v3/direction/*` supply `source:directions` ETA when configured in `providers[]`; otherwise haversine heuristic + deeplinks. Meal legs use the same resolver (capped). `PlaceCard.hours` from Google `regularOpeningHours` / AMAP `opentime_*` when present. |
| Live probe | `make verify-open-meteo-live` — existing healthy `PORT` (default 3010), same pattern as AMAP. Asserts numeric `weather_code` 0–99 and not the fixture triple (`80` + 24/18 °C). WMO `80` is real rain showers; do not treat the code alone as fixture. |
| Rejected | AMAP weather (city adcode only). Google Weather (no CN). |

Rejected weather comparison (why Open-Meteo won): pin-level + global including mainland CN + itinerary-length forecast. AMAP is city-level Chinese copy only. Google Weather has no CN and is a separate billed API; MCP fallback cannot invent CN weather.

### Tripadvisor enrich (HTTP search only)

| Rule | Detail |
| --- | --- |
| Channel | HTTP search body `enrich.tripadvisor`. **Not** an MCP tool argument (ADR-020). |
| Display | Integration guide literal `Tripadvisor.enrich` (dot). JSON path stays `enrich.tripadvisor`. |
| `providers[]` | Do not pass `TRIPADVISOR` for search/details — `errors.capability_unsupported`. Enrich-only. |
| Match | Name + location (ADR-007). Never send Google `place_id` as a Tripadvisor id. |
| Failure | Best-effort; omit enrich fields; do not wipe primary cards. |
| Live search | `GET /locations/nearby?lat=&lon=&radius=1&unit=KM`, header `X-API-Key`. Cluster cards by pin (3 decimals) → one nearby call. Cap nearby concurrency (~3); unbounded parallel bursts return 4xx and skip all enrich. Name score ≥ 0.55. Optional `GET /locations/{terraId}` only after match if rating missing. Payload: `data[].location.traveler_ratings.overall` + `urls.tripadvisor.main`. |
| Implemented | places-agent [`1.places-agent/src/adapters/tripadvisor/`](../1.places-agent/src/adapters/tripadvisor/) — `config.ts`, `direct.ts`, `match.ts`, `card-mapper.ts`, `live.ts`. Fixture remains when `PLACES_VENDOR_MODE` is not `live`. Missing key in live → skip `errors.provider_unconfigured` ([ADR-021](../../adr/ADR-021-live-vendor-no-fixture.md)). |
| Live probe | `make verify-tripadvisor-live` starts a sidecar with `GOOGLE_DIRECT_FORCE_FAIL=0` (Node `--env-file` reloads `.env.local`; `unset` is not enough). Google `textQuery` `"restaurant"` plus an HK pin can return unrelated US POIs — name the area (e.g. Central Hong Kong). |

## Links

- [`3.tech-specs.md`](../../3.tech-specs.md)
- [`places-capabilities.md`](./places-capabilities.md)
- [`../i18n/hk-tw-output.md`](../i18n/hk-tw-output.md)
- [ADR-007](../../adr/ADR-007-tripadvisor-match.md)
- [ADR-020](../../adr/ADR-020-http-only-chat-and-enrich.md)
