---
title: Hotel-name-only geocode can pin the wrong city
type: ops-lesson
status: active
as_of: 2026-09-03
tags:
  - origin
  - geocode
  - amap
  - make_itinerary
related_spec: 1.places-agent/agent-specs/0.refactor-plan.md
related:
  - adr/ADR-048-skeleton-geo-anchor-is-destination.md
  - adr/ADR-042-no-city-encyclopedia-in-source.md
---

# Origin lookup must be destination-bounded

Forward `geocode({ query: hotel name })` with CN locale and AMAP first can return a **different city** than the trip destination. Observed: “Hills Hotel Lisboa” → ~22.19, 113.55 (Macau / Hotel Lisboa), while Lisbon is ~38.72, -9.14.

If that lat/lng is then used as the 80km filter for `make_itinerary`, a valid Lisbon candidate pool is emptied. Stay-only skeletons can still look like HTTP 200 until Feature 83 validates against the **pre-filter** pool.

**Do:** 2play intake step b uses `search_places` with `address` = destination; session stores coords only on an in-city hit. Agent filters on **city** geocode (ADR-048). Far origin coords are dropped, name kept.

**Do not:** hotel-name-only geocode before make; grow a per-city hotel table (ADR-042).
