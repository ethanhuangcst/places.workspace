---
title: Hotel-name-only geocode can pin the wrong city
type: ops-lesson
status: active
as_of: 2026-09-04
tags:
  - origin
  - geocode
  - amap
  - make_itinerary
related_spec: ../../agent-specs/0.refactor-plan.md
related:
  - adr/ADR-048-skeleton-geo-anchor-is-destination.md
  - adr/ADR-042-no-city-encyclopedia-in-source.md
---

# Origin lookup must be destination-bounded

Forward `geocode({ query: hotel name })` with CN locale and AMAP first can return a **different city** than the trip destination. Observed: “Hills Hotel Lisboa” → ~22.19, 113.55 (Macau / Hotel Lisboa), while Lisbon is ~38.72, -9.14.

If that lat/lng is then used as the 80km filter for `make_itinerary`, a valid Lisbon candidate pool is emptied. Stay-only skeletons can still look like HTTP 200 until Feature 83 validates against the **pre-filter** pool.

**Do:** 2play intake step b uses `search_places` with `address` = destination; session stores coords only on an in-city hit. Agent **discover and make** filter on **city** geocode (ADR-048). Far origin coords are dropped (`dropFarOriginCoords`), name kept. Do not geocode the hotel name as the discover 80km anchor.

**Do not:** hotel-name-only geocode before make/discover; grow a per-city hotel table (ADR-042).

**Sign-off 2026-09-04:** `discover_places` still used origin/hotel geocode as the 80km clip. Dual-provider Hangzhou and CN + Macau-coords Lisbon returned **0 places** in ~1s; `make_itinerary` still HTTP **200** with stay+meal-only days. 2play `skeletonIsFillable` then rejected a “successful” make (same class as `e2e-test-results/make-itinerary-issue.md`). After city-anchor discover: Hangzhou 37 / Lisbon 42 places; fetch skeleton had ≥1 attraction/day.

**S6A (2026-09-04):** Intake non-empty origin must resolve to a card **with coords** within 80km of destination geocode. Search fail / no coords / far hits → `not_found`; no silent `degraded` name-only advance.

**S7 (2026-09-04):** Search with `near` = destination coords. Unique exact name coverage auto-hits. Brand-only (凯悦 / Hyatt) and remapped names pause with ≤3 A/B/C. Empty send / skip → **origin coords = destination geocode**. Do not geocode the hotel name as the city gate.

**S7 bug (2026-09-04):** `providersForDestinationText("里斯本")` forced AMAP first; Google place `locationBias` was 5km (meal radius). Brand-only 凯悦 auto-hit skipped ABC. **Do:** geocode city, then `providersForPin`; expand CJK brand to Latin (`hyatt 凯悦`); place bias 50km (Google max); PATCH origin with `destination` from takeoff so discover race cannot empty dest.

**S7 hyatt 500 (2026-09-05):** Live `search_places` “hyatt” + Lisbon **does** return Hyatt Regency Lisbon when agent uses `places_agent`. A Cursor/2play parent `DATABASE_URL=/where2play` made `tsx --env-file` keep the wrong DB → `CallerApiKey` missing (P2021) → every origin search HTTP 500 → UI “找不到”. See `ops/places-agent-local-daemon.md`.

**S7 chip token leak (2026-09-05):** A/B/C chips use `__origin_pick__:N` as the *intake value*, not the hotel name. If that token is stored as `answers.b` / `dailyStart` and sent to `make_itinerary`, stay and the constraints panel show the token; hydrate then skips it → “无起点”. Session PATCH already resolves to `origin_name` + coords. **Do:** persist the candidate name (or `origin_name`); `sanitizeDailyStartName` on validate, POST merge from session, fill/make origin, and hydrate. Never treat the chip id as a stay name.
