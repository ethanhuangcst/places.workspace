---
title: where2play MVP-3 — close record
type: ops-lesson
status: active
as_of: 2026-08-23
tags:
  - where2play
  - mvp-3
  - mode-h
  - e2e
related:
  - ../../workspace-specs/adr/ADR-037-where2play-plan-l2-quanzil.md
  - ../../workspace-specs/adr/ADR-039-cross-product-as-built-vs-target.md
  - 3.where2play/2play-specs/2play-stories.md
---

# where2play MVP-3 — close record (2026-08-23)

## Summary

MVP-3 (**plan-11 / plan-12 / plan-13**) is **Done**. Live E2E (`make test-e2e-mvp3-live`) and fast probe (`e2e/probe_plan_stream.py`) both pass on the dual-server harness. Fast CI remains green (2play 138 tests; agent 478 tests).

## Target pipeline (verified live)

```
discover_places → arrange_day execution=host → OPENAI_CN (stream) → enrich_arrange_transit → expandArrangeDayToSlots → UI
```

## Final status

| Story | Code | `make test` | Live | Story DoD |
| --- | --- | --- | --- | --- |
| plan-11 Mode H | ✓ | ✓ | ✓ London must-see | **Done** |
| plan-13 transit | ✓ | ✓ | ✓ real transit rows | **Done** |
| plan-12 stream | ✓ | ✓ | ✓ full NDJSON path | **Done** |
| MVP-3 batch | — | ✓ | ✓ | **Done** |

## Live evidence (2026-08-23)

- `probe_plan_stream.py`: discovering → arranging → slot_preview → done (~17s).
- `test_mvp3_live.py`: London must-see landmark + transit mode; fail-fast on `plan-error`.

## Root causes fixed (W2d ops)

| Symptom | Fix |
| --- | --- |
| `errors.provider_failed` @ discover (~0.2s) | Agent E2E: `NODE_ENV=development` + `npx tsx --env-file=.env.local server.ts` (not watch dev) |
| E2E 300s timeout on `plan-save` | Upstream discover/plan failure — fixed by agent harness |
| Prisma P2021 `User` table missing | E2E app cmd: explicit `DATABASE_URL=…5435/where2play` via `e2e/run.py` `app_dev_cmd()` |
| Slow staged emit | `PLAN_SLOT_STAGE_MS=0` in `test-e2e-mvp3-live` |

## E2E harness changes

- `3.where2play/e2e/run.py`: shared `e2e_env()` + `app_dev_cmd()`; mvp2/mvp3/chat02 pass `DATABASE_URL`.
- `3.where2play/Makefile`: `test-e2e-mvp3-live` sets `PLAN_SLOT_STAGE_MS=0`.
- Diagnostics: `probe_plan_stream.py`, `probe_agent_routes.py`, `verify_agent_discover.mts`.

## Lessons

- Run `probe_plan_stream.py` before full Playwright when live plan fails — seconds vs minutes.
- **Implemented ≠ Done** when AC includes live honesty; batch Done only after W2d E2E.
- One story at a time to DoD; do not bundle 31+33+32 + ops in one signoff.
- Reusing stale `:3030` / `:3010` via `with_server` can hide env drift — kill listeners when debugging.

## Follow-on (MVP-4)

Story **24** `chat-02` closed same day: `make test-e2e-chat02` (local draft refresh + logout clear). Next: **23** `chat-01` live OPENAI_CN + `itineraryPatch`.
