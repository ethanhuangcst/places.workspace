---
title: what2eat MVP-4 lessons
type: ops-lesson
status: active
as_of: 2026-08-20
tags:
  - what2eat
  - mvp-4
  - e2e
  - decide
  - chat
related_spec: 2.what2eat/2eat-specs/2eat-stories.md
related:
  - adr/ADR-029-decide-criteria-draft-hydrate.md
  - knowledge/web-app-development/what2eat-mvp3-lessons.md
  - knowledge/web-app-development/what2eat-decide-locale-draft.md
  - knowledge/maps/price-level-live.md
---

# what2eat MVP-4 — lessons

Captured during MVP-4 (sort, chat UX polish, price UI, criteria drafts, panel size) close on 2026-08-20.

## What shipped

- **decide-08 / decide-03:** Sort by rank/rating/distance/price; reshuffle re-queries vendor and resets rank (verified existing suites).
- **chat-02 / chat-03:** NW napkin-corner resize grip; rich `blocks[]` hydrate (verified).
- **chat-04:** Shared pending bubble (`data-testid="chat-pending"`, `eat.chat.pending`) + transcript auto-scroll in list and place composers.
- **decide-09:** Map `price_level` / `price_per_person` → pick card (`pick-price`) and details fact row (`details-price`); honest unavailable when missing.
- **decide-10 / decide-11:** `sessionStorage` drafts via `decide-draft.ts`; hydrate **URL → draft → SearchCache → profile (virgin only)** ([ADR-029](../../adr/ADR-029-decide-criteria-draft-hydrate.md)).
- **chat-05:** Persist `{width,height}` under `w2e.chat.panelSize`; clamp with existing mins; logout clears via `w2e.chat.` prefix.

## Implementation notes

| Topic | Guidance |
| --- | --- |
| Profile vs draft | Never `setLocation(defaultLocation)` (or meal/budget/craving) when draft exists or field is touched — `mayApplyProfileDefault`. |
| Meal default | `defaultMealContextSelection()` is a **function**; calling it as a value breaks types and hydrate. |
| Pending UX | One shared pending row in `chat-composer` when `busy`; both panels already pass `busy`. |
| Panel size | Load on mount, save on resize end; unit tests need a `localStorage` stub under Node. |
| Vitest JSX | Include `tests/**/*.test.tsx` and use jsdom for composer pending tests. |

## Live E2E

`make test-e2e-mvp4-live` (`e2e/test_mvp4_live.py`): sort, reshuffle, chat open/resize/rich, `pick-price` (band or unavailable), locale location draft, panel size → localStorage, `chat-pending` then agent message.

**Recent live pass:** 2026-08-20 (`mvp4 live journey ok`; agent `:3010`, app `:3020`).

**Unit gate:** `make test` — 149 cases green at close.

## ADRs

- [ADR-029](../../adr/ADR-029-decide-criteria-draft-hydrate.md) — Decide criteria draft hydrate (no new ADR for panel size / pending; storage conventions match existing chat keys).

## Operator checklist

1. Postgres `:5435` (`make up` in what2eat).
2. places-agent + what2eat on `:3010` / `:3020` (or let `e2e/run.py` start them).
3. `make test` + `make test-e2e-mvp4-live` before DoD sign-off.

**DoD status:** Implementation + automated gates green; **awaiting user usability confirm** before marking stories slice fully signed off.

## Post-close follow-ups

See [what2eat-mvp4-followups.md](./what2eat-mvp4-followups.md) (SSR drafts, list chat key, chat timeout, provider strip, ADR-031 empty-AMAP fallback, card-first hydrate).
