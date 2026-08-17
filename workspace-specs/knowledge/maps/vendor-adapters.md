---
title: Map and weather vendor adapter notes
type: research-note
status: active
as_of: 2026-08-17
tags:
  - maps
  - amap
  - google
  - open-meteo
  - mcp
related_spec: workspace-specs/3.tech-specs.md
related:
  - knowledge/handbook.md
  - knowledge/maps/places-capabilities.md
  - knowledge/i18n/hk-tw-output.md
  - adr/ADR-005-caller-driven-providers.md
  - adr/ADR-014-open-meteo-weather.md
  - adr/ADR-017-gmaps-mcp-fallback.md
---

# Map and weather vendor adapter notes

Reusable integration facts for places-agent adapters. **Decisions:** [ADR-005](../../adr/ADR-005-caller-driven-providers.md) (caller `providers[]`), [ADR-014](../../adr/ADR-014-open-meteo-weather.md) (Open-Meteo only), [ADR-017](../../adr/ADR-017-gmaps-mcp-fallback.md) (Google direct then Worker MCP). No secrets in this file.

## Summary

AMAP Web 服务 uses **lng,lat** and GCJ-02 in CN/HK/MO/TW. Google Places is WGS-84; when direct REST fails, use the Cloudflare Worker MCP and still tag `GOOGLE_MAPS`. Weather is Open-Meteo (WMO codes), not AMAP/Google weather. Open-Meteo has no language parameter — translate `weather.wmo.{code}`.

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
| Not | Tripadvisor replacement. Not a geo-forced default (ADR-005). Do not call AMAP `weatherInfo` (ADR-014). |
| Trust | Key only on places-agent. Deeplinks secret-free. |

### Google Maps (direct + Worker MCP)

| Rule | Detail |
| --- | --- |
| Direct | `maps.googleapis.com` + `GOOGLE_MAPS_API_KEY` on places-agent. Places API (New), Geocoding, Routes. CRS WGS-84. |
| Fallback | On egress failure only: Streamable HTTP MCP `GMAPS_MCP_URL` + `GMAPS_MCP_BEARER`. `tools/list` first. |
| Provenance | Always `GOOGLE_MAPS`. Do not invent provider id `GMAPS_MCP`. |
| Keys | Worker holds the Maps key. Agent sends MCP bearer only. No `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY`. |
| Do not | Fall back to AMAP unless the caller asked for AMAP. Worker does not fix mainland **client** Google Maps app/web (ADR-006). |
| Weather | Do not enable Google Weather API. No mainland CN coverage. |

### Open-Meteo (weather)

| Rule | Detail |
| --- | --- |
| Call | `GET /v1/forecast` with `latitude` + `longitude`. Not a `providers[]` vendor. |
| Free vs commercial | Free `api.open-meteo.com` is non-commercial. Production: `customer-api.open-meteo.com` + `OPEN_METEO_API_KEY`. Attribution required (CC BY). |
| Language | **No** `language` param. Body is numeric (`weather_code`, °C, %, m/s). Docs are English. |
| Display | Map `weather_code` → `weather.wmo.{code}` in `EN` / `CN` / `HK` / `TW`. Format numbers with `Intl`. LLM narrative uses localized strings — never English “Slight rain showers” in CN/HK/TW. |
| Rejected | AMAP weather (city adcode only). Google Weather (no CN). |

Rejected weather comparison (why Open-Meteo won): pin-level + global including mainland CN + itinerary-length forecast. AMAP is city-level Chinese copy only. Google Weather has no CN and is a separate billed API; MCP fallback cannot invent CN weather.

## Links

- [`3.tech-specs.md`](../../3.tech-specs.md)
- [`places-capabilities.md`](./places-capabilities.md)
- [`../i18n/hk-tw-output.md`](../i18n/hk-tw-output.md)
