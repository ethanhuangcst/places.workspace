# ADR-071: Descope in-page chat refine — replan only

Family backlog: [`product-backlog.md`](../product-backlog.md)

## Status

**Accepted**（2026-09-18）

## Context

MVP-T9 shipped `plan_trip` refine + where2play `/api/chat` (ADR-050, ADR-070). Live probes (Shanghai family) showed:

- **Probe 1** (vague「改近一点」): clarify works — good UX for true-agent, but product chose simpler scope.
- **Probe 2** (slot-complete morning replace): **unstable** — search empty, false success copy, full `days[]` rewrite noise (`refill_day_indices` bloated).

Product decision: **remove chat-based itinerary edits**; travelers who want a different trip use **Replan** (confirm → new full-loop). Planning **need_input** composer (hotel, expand_radius, etc.) stays.

## Decision

### D1 — Cancel T9 in-page refine (full-stack)

Remove:

- where2play post-complete composer + refine thread + `POST /api/chat` refine
- places-agent `plan_trip` refine mode (`plan-trip-refine.ts`, `refine` body field)

Keep:

- Takeoff → T3 assistant progress → fill → complete
- **Replan** (`ReplanDialog`, soft replan chip, `play.plan.replan*`)
- **need_input** composer during planning (not after complete)

### D2 — Supersede ADR-070

[ADR-070](./ADR-070-refine-skeleton-true-agent-loop.md) **Cancelled** — skeleton refine loop not pursued.

### D3 — what2eat unchanged

2eat list/place chat and `/api/chat` on what2eat are out of scope.

### D4 — Saved chat snapshot (T10 / `2play-plan-25` AC2–3)

**Done · usable Confirmed 2026-09-21。** 快照不是 refine transcript（已取消）。

- **对话快照** = 保存瞬间的 **Plan 助手线程**（intake 用户答 + 助手进度句 + 可选 system 分隔）。
- **`SavedItinerary.tripId`** = session `criteria.tripId`（有则必传），供详情 `fetch_trip_details` `artifacts`；`snapshot` 仍为 `ItineraryDto`，不含 visa/tips 政策（ADR-046）。
- 每次保存新建一行；未再次保存则旧行不变。

UI 只读 transcript → `2play-saved-26`；与 Plan 完成态同构 → `2play-plan-37` / 24-P1b。

## Consequences

**Positive**

- Simpler product: one path to change trip (replan).
- Less LLM cost and failure surface on post-complete chat.
- UI matches capability: no composer promising edits that are removed.

**Negative**

- No incremental tweak without full replan (user cost).
- Removed agent refine code; reintroducing later needs new story + ADR.

## Implementation（2026-09-18）

Full-stack descope landed in `where2play` + `places-agent`:

- Removed post-complete composer, refine thread, `POST /api/chat`, agent `plan_trip` refine.
- Kept Replan dialog + need_input composer during planning.
- Vitest regression green for thread ordering, i18n catalog, T3 hydrate, no `plan-nav-input` after complete.

## Verification

| Gate | Status |
| --- | --- |
| Unit / component (vitest) | **Pass** — descope-related suites |
| Manual browser (takeoff → plan → complete → Replan) | Product **usable Confirmed** 2026-09-20 |
| DoD usable confirm | **Done** — 2026-09-20 |

Descope is **Done** (usable Confirmed).

## References

- Probe: [`shanghai-family-refine-probe.md`](../knowledge/agent/shanghai-family-refine-probe.md)
- Close notes: [`where2play-t9-close-lessons.md`](../knowledge/web-app-development/where2play-t9-close-lessons.md)
- Supersedes: ADR-070, MVP-T9 refine stories
