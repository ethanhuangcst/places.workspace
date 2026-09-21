---
title: Restart where2play after Prisma schema changes
type: ops-lesson
status: active
as_of: 2026-09-21
tags:
  - prisma
  - where2play
  - saved-itinerary
related_spec: specs/2play-specs/2play-stories.md
related:
  - knowledge/agent/plan-return-hydrate-artifacts.md
---

# Restart where2play after Prisma schema changes

## Summary

Adding `SavedItinerary.tripId` and applying the migration is not enough while `next dev` is already running. The process keeps the Prisma Client generated before the field. `POST /api/saved` then returns 500 (`Unknown argument tripId`) and 我的行程 stays empty even though the column exists in Postgres.

## Evidence

- Shanghai 3-day save, 2026-09-21: migration `20260921092558_saved_itinerary_trip_id` was applied; the Next process on :3030 had started on 2026-09-20.
- Log: `PrismaClientValidationError: Unknown argument tripId` at `savedItinerary.create`. `GET /api/saved` stayed 200.
- `npx prisma generate` plus a restart of `npm run dev` on :3030 cleared it. Vitest had already passed because its setup runs `prisma migrate deploy` against a fresh client.

## Lesson / guidance

After a Prisma schema change in where2play: generate the client, apply the migration to the app database, then restart the dev server that was started before generate. A green API test does not prove the long-running Next process loaded the new client.

## Links

- Story: `2play-plan-25`
