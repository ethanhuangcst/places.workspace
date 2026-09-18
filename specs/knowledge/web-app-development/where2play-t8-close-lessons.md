# where2play MVP-T8 close — E2E drift and assistant thread UX

**Status:** accepted · 2026-09-18  
**Scope:** MVP-T8 Batch 1（103/104 · TD-8/9/10 · 93f/90f）

## E2E drift after T3 / takeoff-11

Product flow moved to takeoff-11 + submit confirm + `plan_trip` skeleton/fill + `PlanAssistantNav`. Legacy E2E assumed:

- Partial takeoff fields only
- Mode H L2 arrange + `plan-slot-preview`
- Separate `PlanChatPanel` on plan page

**Fix:** Shared `e2e/takeoff_helpers.py` (`fill_takeoff_and_confirm`); rewrite `mvp3-live` / `mvp10-structure`; defer `chat02` to MVP-T9 (composer in nav). Geocode for structure gate: prefer CN locale + 杭州 when EN Lisbon geocode is flaky in CI.

## Assistant thread: post-complete affordances

`assistant_next_hint` and soft replan must not appear at skeleton ready — only after `planCompleteLine` (fill done). Gate in `plan-page.tsx`; DOM order in `plan-assistant-nav.tsx`: complete bubble → hint → replan chip.

## Open after T8

- 12-case skeleton probe test12 台北 — known fail; separate quality story
- chat-02 E2E — rewrite in MVP-T9 against `plan-nav-input` + localStorage
- Transit meal slot ids (`lunch`) — localize at render via `transitEndpointLabel`

## No new ADR

Mechanisms covered by ADR-061/067/068/069 and existing T5 plan decisions.
