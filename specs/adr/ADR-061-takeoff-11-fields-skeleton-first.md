# ADR-061 — Takeoff 11 fields; skeleton-first plan_trip; agent-driven needs_input

- **Status:** Accepted (T2 D1–D4/D8/D9 + geocode). **T3+ delivery split superseded by [ADR-062](./ADR-062-mvp-t3-skeleton-vs-t4-nominate.md)** (2026-09-10).
- **Date:** 2026-09-09
- **Deciders:** product + agent
- **Scratch:** disposed — see stub [`../agent-specs/tmp-0909.md`](../agent-specs/tmp-0909.md); durable SoT = this ADR + [`agent-design.md`](../agent-specs/agent-design.md)
- **Work plan:** [`../plan.md`](../plan.md)
- **Stories:** `2play-plan-100` · `agent-geocode-100` (T2 Done); T3/T4 see ADR-062

## Context

MVP-T1 as-built: 8 takeoff fields + 4 fixed agent `need_input` questions (`hotel`, `start_time`, `must_see`, `other`). Product wants:

1. All boundary inputs on the takeoff bar (11 fields; drop must-see from the bar).
2. Trip assistant no longer runs the fixed 4-question intake; it becomes conversation + progress after submit.
3. `plan_trip` creates a **skeleton** first (not the full fill on takeoff submit).
4. `needs_input` remains for **agent-driven** questions and third-party MCP clients (e.g. ChatBox), not a fixed where2play questionnaire.

Delivery is **split** so takeoff UX can ship without blocking on plan_trip/assistant changes:

| MVP | Scope |
| --- | --- |
| **MVP-T2** | Takeoff 11 fields through **submit** (validation + layout + i18n; geocode structured labels) — **Done** |
| **MVP-T3 / T4+** | **See [ADR-062](./ADR-062-mvp-t3-skeleton-vs-t4-nominate.md)** (T3 = takeover + skeleton + debug; T4 = nominate + chat refine) |

## Decision

1. **Eleven takeoff fields:** destination, tripType, budget, startDate, days, partySize, pace, transit, startTime, origin, other. No must-see on takeoff.
2. **Blur validation:** destination via extended `geocode` `{ country, city, city_en? }`; origin keeps suggest_places 100%/partial/no-match popup; `startTime` = daily default for all days.
3. **After submit:** skeleton-first `plan_trip`; no fixed four-question where2play intake. **Packaging of T3 vs T4 (nominate/chat) → ADR-062.**
4. Stories: `2play-plan-100` (T2 Done). Later stories per ADR-062 / backlog.

## Consequences

- T2 shipped 2026-09-09 (usable Confirmed).
- Geocode contract grows structured admin fields (backward-compatible optional).
- MCP callers keep `needs_input`; where2play takeoff no longer depends on the fixed four prompts.
- Mobile takeoff layout deferred to implementation design (D9).

## Alternatives considered

| Option | Why not |
| --- | --- |
| Keep 4 fixed questions after 11-field takeoff | Duplicates hotel/start_time/other; fights “pure chat” assistant |
| Full plan on takeoff submit | Too slow / couples fill to form; product wants skeleton first |
| Ship takeoff + plan_trip in one MVP | Violates incremental delivery; product split T2/T3 |

## Related

- ADR-042, ADR-050, ADR-055, ADR-060, **ADR-062**
- [`agent-design.md`](../agent-specs/agent-design.md)
- Backlog: `2play-plan-100` (Done); T3+ per ADR-062
