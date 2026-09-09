---
title: where2play takeoff / constraints mock-first sync
type: design-direction
status: active
as_of: 2026-09-09
tags:
  - where2play
  - mockup
  - takeoff
  - plan-constraints
  - i18n
related_spec: specs/2play-specs/2play-design.md
related:
  - adr/ADR-061-takeoff-11-fields-skeleton-first.md
  - 2play-specs/ui-mockup/06-plan-takeoff-11.html
  - 2play-specs/ui-mockup/06-plan-qa.html
---

# where2play takeoff / constraints mock-first sync

## Summary

MVP-T2 UI polish shipped by locking HTML mocks first, then updating design/stories/tests, then porting CSS + i18n + components so the app matches the mock SoT. Shared CSS column tracks (`--takeoff-cols`, `--constraint-cols`) are what keep multi-row field alignment honest.

## Evidence

- Mock SoT: `06-plan-takeoff-11.html` (two rows, seven tracks) and `06-plan-qa.html` (11 constraint cells; no must-see row).
- User iterates copy/layout on mocks; only after confirm, code follows.
- Takeoff vs constraints **label strings diverge on purpose** (e.g. takeoff「天数」vs constraints「行程天数」); use separate i18n keys (`trip_days` vs `constraint_days`).
- When the user asks for text-only renames with unchanged x/y, do **not** widen `--constraint-label-w` — longer labels may clip/overflow inside the fixed label column.

## Lesson / guidance

1. Treat `ui-mockup/` as pixel/DOM SoT for Plan takeoff and constraints; sync `where2play/app/mockup*.css` from `specs/2play-specs/ui-mockup/assets/` when layout locks.
2. Prefer shared track variables over separate 3-col / 4-col grids that drift.
3. Keep must-see off takeoff and off the T2 constraints panel (ADR-061 / ADR-042).
4. Order of delivery that worked: mock → `2play-design` page contract → stories/AC + test matrix → i18n + components → vitest on labels/structure.

## Links

- ADR-061, `2play-plan-100`, design §4.2.1 / §4.7
