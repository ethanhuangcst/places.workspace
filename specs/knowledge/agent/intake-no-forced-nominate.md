---
title: Intake no forced nominate
type: design-direction
status: active
as_of: 2026-09-09
tags:
  - plan_trip
  - must_see
  - intake
related_spec: specs/agent-specs/agent-design.md
related:
  - adr/ADR-060-intake-no-forced-nominate.md
  - agent/nominate-must-see-prompt-probe.md
---

# Intake: do not force L3 nominate for takeoff chips

## Summary

Takeoff `plan_trip` intake can produce must-see chips via the tool loop (`geocode` → `search_places` → `commit_trip`). Guaranteeing chips with a pre-loop `nominateMustSeeViaLlm` (L3) was rejected: purity of the ring beats empty-Q3 UX risk. Empty chips are product-legal (ADR-060); Q3 offers hand-type + refetch.

## Evidence

- Forced nominate masked empty pools and mixed L3 (plan_itinerary path) into takeoff.
- Lisbon usable sign-off (2026-09-09): few or zero chips still accepted when 8 takeoff fields + 4 questions worked.
- L3 `buildNominateMustSeeUserMessage` remains for `plan_itinerary` / probes only.

## Lesson / guidance

- Prefer loop-collected chips; do not reintroduce `runForcedNominate` on intake.
- UI must handle empty `must_see.options` (i18n empty state, hand input, optional candidates refetch) without a third automatic `plan_trip`.
- Full-loop / T2 may still nominate when scheduling — that is not takeoff guarantee.

## Links

- [ADR-060](../../adr/ADR-060-intake-no-forced-nominate.md)
- [agent-design MVP-T1 as-built](../../agent-specs/agent-design.md)
- [mvp-1t-closing-plan](../../mvp-1t-closing-plan.md)
