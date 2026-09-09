# ADR-061 — Takeoff 11 fields; skeleton-first plan_trip; agent-driven needs_input

- **Status:** Accepted (T2 decisions D1–D4/D8/D9 + geocode shape); T3 clauses remain pending implementation
- **Date:** 2026-09-09
- **Deciders:** product + agent
- **Scratch:** [`../agent-specs/tmp-0909.md`](../agent-specs/tmp-0909.md)
- **Work plan:** [`../plan.md`](../plan.md)
- **Stories:** `2play-plan-100` · `agent-geocode-100` (T2); `2play-plan-101` · `agent-itinerary-100` (T3)

## Context

MVP-T1 as-built: 8 takeoff fields + 4 fixed agent `need_input` questions (`hotel`, `start_time`, `must_see`, `other`). Product wants:

1. All boundary inputs on the takeoff bar (11 fields; drop must-see from the bar).
2. Trip assistant no longer runs the fixed 4-question intake; it becomes conversation + progress after submit.
3. `plan_trip` creates a **skeleton** first (not the full fill on takeoff submit).
4. `needs_input` remains for **agent-driven** questions and third-party MCP clients (e.g. ChatBox), not a fixed where2play questionnaire.

Delivery is **split** so takeoff UX can ship without blocking on plan_trip/assistant changes:

| MVP | Scope |
| --- | --- |
| **MVP-T2** | Takeoff 11 fields through **submit** (validation + layout + i18n; geocode structured labels) |
| **MVP-T3** | After-submit: assistant takeover + `plan_trip` skeleton-first + nominations + agent-driven `needs_input` |
| **MVP-T4+** | Former T2–T5 (pool / day fill / meals / tips / chat) renumbered in [`plan.md`](../plan.md) |

## Decision

1. **Eleven takeoff fields:** destination, tripType, budget, startDate, days, partySize, pace, transit, startTime, origin, other. No must-see on takeoff.
2. **Blur validation:** destination via extended `geocode` `{ country, city, city_en? }`; origin keeps suggest_places 100%/partial/no-match popup; `startTime` = daily default for all days.
3. **After submit (T3):** `plan_trip` lays skeleton only; no fixed four questions; agent may `ask_user` when needed; auto-nominate must-sees via `search_places` **with reasons**; user confirms/edits in chat (ADR-042, ADR-060).
4. **UI (T3):** assistant shows progress + skeleton key info; full plan details on 行程规划 section.
5. Stories: `2play-plan-100` (T2), `2play-plan-101` + agent plan_trip story (T3). Supersedes T1 as-built intake UX for where2play once T2+T3 land.

## Consequences

- T1 8+4 path remains as-built until T2/T3 ship.
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

- ADR-042, ADR-050, ADR-055, ADR-060
- [`agent-design.md`](../agent-specs/agent-design.md) (T2 target section TBD when implementing)
- Backlog: `2play-plan-100`, `2play-plan-101`
