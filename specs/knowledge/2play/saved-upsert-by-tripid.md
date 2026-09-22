---
title: Saved itinerary — upsert by tripId (not version history)
type: ops-lesson
status: active
as_of: 2026-09-22
tags:
  - where2play
  - saved
  - tripId
---

# Saved itinerary — upsert by tripId

## Lesson

Story 25 originally required **one new `SavedItinerary` row per Save click** (explicit “no upsert”). That matched a “snapshot history” model but conflicted with bookmark UX: users expect one card per ongoing Plan trip.

## Product rule (current)

- Same `(userId, tripId)` → **overwrite** snapshot + messages (`POST /api/saved`).
- Missing `tripId` → still **create** (multiple nulls allowed).
- Replan that yields a **new** `tripId` → new card; old card unchanged.

## Spec pointers

- `2play-stories` Story 25 AC2/AC3 (amended 2026-09-22)
- `2play-design` §2.4.5
- ADR-071 D4 amendment
- Prisma `@@unique([userId, tripId])`
