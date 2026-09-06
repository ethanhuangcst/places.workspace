# ADR-014: Open-Meteo for weather; localize English/WMO output

## Status

Accepted

## Context

Itinerary and “go-out” copy need weather at a place pin. Alternatives compared in [`../3.tech-specs.md`](../3.tech-specs.md): Open-Meteo, AMAP 天气 (`adcode` only), Google Maps Platform Weather API (no mainland CN coverage). Open-Meteo forecast JSON is **not** locale-aware: `weather_code` is a WMO integer; docs and any textual interpretation are English. Place vendors (Google/AMAP) can return names in `languageCode`; Open-Meteo cannot. Passing English condition strings into `CN` / `HK` / `TW` output would break ADR-011.

## Decision

1. **Open-Meteo is the only weather provider** for this family. Do not call AMAP `weatherInfo` or Google Weather API for product weather.
2. Call `GET /v1/forecast` from places-agent (lat/lng). Commercial deploy uses `customer-api.open-meteo.com` + `OPEN_METEO_API_KEY` and CC BY attribution.
3. Treat Open-Meteo fields as **machine facts**, not display copy:
   - Keep `weather_code` (and numeric temp/wind/humidity) in the payload.
   - Map each code to i18n key `weather.wmo.{code}` in catalogs `EN`, `CN`, `HK`, `TW`.
   - Format numbers/units with locale formatters (ADR-011 Layer C).
4. **Do not** pass Open-Meteo English descriptions through as vendor facts (that Layer B exception is for place names/addresses only).
5. LLM itinerary/chat narrative must use **already-localized** condition strings (or the code + locale catalogs). Do not paste English “Slight rain showers” into `CN` / `HK` / `TW` prose.

## Rationale

- Open-Meteo is global (including mainland CN), pin-level, and matches itinerary days. AMAP weather is city-level adcode; Google Weather has no CN coverage.
- Open-Meteo has no `language` parameter; localization is our job.
- HK vs TW weather wording differs (e.g. 驟雨 vs 陣雨); catalogs, not OpenCC.

## Consequences

- Four catalog packs include the WMO code table from the first weather-facing story.
- Tests assert `weather.wmo.*` keys and locale wording — not English-only condition strings as the contract (except `EN`).
- Missing code → `EN` catalog → key `weather.wmo.{n}`; never fail the whole itinerary solely for a missing weather string.
- Follow-up: Feature 13 AC covers weather labels; implementation must not treat Open-Meteo text as Layer B.

## Date

2026-08-17
