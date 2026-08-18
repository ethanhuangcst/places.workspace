---
title: Places API capability matrix
type: research-note
status: active
as_of: 2026-08-17
tags:
  - maps
  - amap
  - google
  - tripadvisor
related_spec: specs/1.req-specs.md
related:
  - knowledge/handbook.md
  - knowledge/maps/vendor-adapters.md
  - adr/ADR-005-caller-driven-providers.md
  - adr/ADR-017-gmaps-mcp-fallback.md
---

# Available capabilities — Places APIs

Probe date: 2026-08-14 (Google Places API New re-confirmed OK).  
Scope: AMAP, Google Maps, Tripadvisor Terra. No secrets in this file.

Consolidated guidance (selection, nav, enrichment): [`../handbook.md`](../handbook.md) §§3–4.  
Adapter gotchas (AMAP lng,lat / GCJ-02, Google Worker MCP, Open-Meteo WMO): [`vendor-adapters.md`](./vendor-adapters.md).

| # | Capability | Example | AMAP | Google Map | Tripadvisor |
| --- | --- | --- | --- | --- | --- |
| 1 | Text / keyword place search | “hotpot in Shanghai”, “ramen in Tokyo” | Yes | Yes | Yes |
| 2 | Nearby search by lat/lng | Restaurants within 500 m of a pin | Yes | Yes | Yes |
| 3 | Search restaurants | Filter dining / `RESTAURANT` | Yes | Yes | Yes |
| 4 | Search attractions / places | Museums, parks, POIs | Yes | Yes | Yes |
| 5 | Place details by id | Name, address, coords after pick | Yes | Yes | Yes |
| 6 | Geocode (address → coords) | “People’s Square, Shanghai” → lat/lng | Yes | Yes | No |
| 7 | Reverse geocode | Lat/lng → address | Yes | Yes | No |
| 8 | Navigate / directions | Drive route A → B, ETA, distance | Yes | Yes | No |
| 9 | Rating | 4.5 stars / traveler score | Yes | Yes | Yes |
| 10 | Price signal | Avg cost / price level | Yes | Yes | No |
| 11 | Opening hours | Open now / weekly periods | Yes | Yes | Yes |
| 12 | Phone number | Call the venue | Yes | Yes | Yes |
| 13 | Website / official URL | Venue home page | Yes | Yes | Yes |
| 14 | Photos | Cover or gallery | Yes | Yes | Yes |
| 15 | Review snippets / reviews API | Featured or full reviews | Partial | Yes | Yes |
| 16 | Category / type tags | Restaurant, attraction, hotel | Yes | Yes | Yes |
| 17 | Distance from search center | “320 m away” | Yes | Compute | Yes |
| 18 | Map / deep links | Open in map app, directions URL | Yes | Yes | Yes |

**Yes** = supported by that provider’s APIs (probed or documented).  
**Partial** = field exists but often sparse or limited.  
**No** = not a primary capability of that provider.

The **Google Map** column is the `GOOGLE_MAPS` adapter contract. Transport may be direct REST or Cloudflare Worker MCP after egress failure ([ADR-017](../../adr/ADR-017-gmaps-mcp-fallback.md)). Capabilities do not change; provenance stays `GOOGLE_MAPS`.

## Provider selection

Caller-driven: apps pass `providers[]` (and optional enrich/merge). places-agent does **not** force mainland → AMAP. See handbook §4 and [ADR-005](../../adr/ADR-005-caller-driven-providers.md). `GMAPS_MCP` is not a `providers[]` id.
