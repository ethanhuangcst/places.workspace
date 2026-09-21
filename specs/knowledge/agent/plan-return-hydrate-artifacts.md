---
title: Plan return must re-fetch artifacts for tips
type: ops-lesson
status: active
as_of: 2026-09-21
tags:
  - travel-tips
  - hydrate
  - artifacts
  - plan-session
related_spec: specs/2play-specs/2play-stories.md
related:
  - adr/ADR-046-fetch-only-visa-display.md
  - adr/ADR-073-plan-session-draft-itinerary.md
  - adr/ADR-075-tips-ndjson-via-fill-poll.md
---

# Plan return must re-fetch artifacts for tips

## Summary

Navigating 我的行程 → 行程规划 restores itinerary from `PlanSessionCache` + skeleton/filled, but tips/visa live only in trip `artifacts`. Without `fields` including `artifacts` and a sibling `travelTips` on `GET /api/plan/current`, the panel stays empty even when the ledger still has tips.

## Evidence

- Root cause for `2play-plan-106`: `refreshItineraryFromTripLedger` historically used `fields: ["skeleton","filled"]` only; plan-page hydrate never called `setTravelTips`.
- Policy must not be copied into `itineraryJson` (ADR-046 / ADR-073). React memory alone is not enough for return paths.
- Saved detail has no `tripId` column yet (Save P2). Bridge: if session `criteria.tripId` exists and destination matches the saved row, fetch `artifacts` the same way; otherwise stay honest-empty.

## Lesson / guidance

1. Any return/hydrate path that should show tips or visa must `fetch_trip_details` with `artifacts` and map via `travelTipsPayloadFromSlice`.
2. Pass tips as a JSON sibling (`travelTips`), never merge visa/policy into `itineraryJson`.
3. Do not treat NDJSON `tips` events as durable across navigations; re-read the ledger.
4. Persisting `tripId` on `SavedItinerary` belongs to Save P2 — do not invent a Prisma column inside a hydrate story.

## Links

- Story: `2play-plan-106`
- Design §4.10 data-flow table (tips/visa / 完成 rows)
