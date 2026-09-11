# ADR-062 — MVP-T3 skeleton UX vs T4 nominate/chat refine

- **Status:** Accepted（T3/T4 **delivery split** still holds）· discovery relationship **superseded by** [ADR-067](./ADR-067-llm-driven-discovery-replaces-stops-pool.md) for LLM-driven discovery
- **Date:** 2026-09-10
- **Deciders:** product + agent
- **Supersedes (delivery split only):** [ADR-061](./ADR-061-takeoff-11-fields-skeleton-first.md) clauses that bundled nominations + chat refine into MVP-T3
- **Superseded in part by:** [ADR-067](./ADR-067-llm-driven-discovery-replaces-stops-pool.md) (T3 internal discovery uses nominate capability; T4 still user-facing must-see)
- **Work plan:** [`../plan.md`](../plan.md)
- **Stories:** `2play-plan-101` · `agent-itinerary-100` (T3, **Done**); T3++ `agent-discover-110*`; `2play-plan-102` · `agent-itinerary-101` (T4, after T3++)

## Context

ADR-061 Accepted T2 (Takeoff 11 → submit) and sketched T3 as: assistant takeover + `plan_trip` skeleton-first + must-see nominations with reasons + agent-driven `needs_input` / chat refine.

After T2 usable confirm, product wants a thinner next slice: **see a trip id, progress copy, and a skeleton on the existing Plan UI** before investing in nomination quality and open chat. Hotel / daily start time / other already live on takeoff; must-see is **not** a pre-filled takeoff field — so the T1 fixed **four-question intake** must not run after submit.

## Decision

### Delivery split

| MVP | Scope |
| --- | --- |
| **MVP-T3** | Specs/chat-dump cleanup · assistant takeover after takeoff submit · `plan_trip` creates trip (`trip_id`) · progress messages in assistant (phase → i18n) · skeleton from takeoff inputs · render skeleton with **as-built** Plan UI · **reuse `/debug/plan`** for plan info + stops-pool observability |
| **MVP-T4** | Recommend must-see with reasons · chat to gather more input and refine skeleton |
| **MVP-T5+** | After skeleton confirm: `trip_details` / progressive fill stop-by-stop · meals · tips · chat-edit · probes (renumber former T4–T7) |

### T3 product rules

1. **No fixed last-4-question intake** on where2play after submit (hotel / start_time / other already on takeoff; must-see deferred to T4).
2. **`plan_trip` skeleton-first:** stop after make/commit skeleton — no `plan_next_stop` fill, meals, or directions in T3.
3. **Progress UX:** BFF/agent phase events mapped to i18n strings in `plan-nav` (ADR-050: no product LLM narrator in 2play).
4. **Debug:** keep and reuse [`/debug/plan`](../../where2play/app/(app)/debug/plan/page.tsx) (`plan-debug-page`) to show current trip / boundaries and city **stops-pool** while developing and accepting T3.
5. **Cleanup (Story 0):** **Done 2026-09-10** — chat dumps + `mvp-1t-closing-plan` → `specs/archive/`; `tmp-0909` stubbed; links → durable SoT.
6. **UI copy:** user-facing Chinese uses **框架**; internal contracts keep `skeleton` / 骨架 — see `2play-design` §4.7.1 and knowledge `ui-framework-vs-skeleton.md`.

### What ADR-061 still owns

T2 field list, blur/geocode shape, and the principle that `plan_trip` is skeleton-first remain **Accepted**. Only the **T3+ packaging** moves here.

## Consequences

- Backlog rows for T3 drop “必去提名 / needs_input chat”; T4 gains nominate + chat refine stories.
- Former “补池 + 按日 filled” etc. shift to T5+.
- MCP / ChatBox may still use agent-driven `needs_input` later; where2play T3 does not ship a fixed questionnaire.
- ADR-worthy follow-up only if skeleton commit contract or phase-event schema needs a separate decision.

## Alternatives considered

| Option | Why not |
| --- | --- |
| Keep nominations inside T3 | Couples skeleton UX to nominate quality; blocks usable demo |
| Re-run 4-question intake after 11-field takeoff | Duplicates three fields; must-see should not be a forced pre-fill (ADR-060) |
| New debug UI for pool | Existing `/debug/plan` already covers trip + stops-pool |

## Related

- ADR-042, ADR-050, ADR-055, ADR-060, ADR-061
- [`plan.md`](../plan.md) · [`product-backlog.md`](../product-backlog.md)
- Stories: `2play-plan-101` · `agent-itinerary-100`（AC Ready）
- Design: [`2play-design.md`](../2play-specs/2play-design.md) §4.7.1 · [`agent-design.md`](../agent-specs/agent-design.md) MVP-T3（含时序图）
- Tests: `2play-test-plan` §13 · `agent-test-plan` §40
