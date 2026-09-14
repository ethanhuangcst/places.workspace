# Full-loop early stop — A+B prompt fix (MVP-T5 S1)

**as_of:** 2026-09-14  
**Status:** Done (probe)

## Problem

`plan_trip` full loop (`skeleton_only=false`) called `plan_next_stop` ~3 times then `stop`, marking `ready` at ~17–20% fill.

## Fix (A+B — no code hard loop)

- **A:** `FULL_LOOP_STOP_TOOL_DESCRIPTION` — stop only after `trip_complete`.
- **B:** `buildFullLoopSystemPrompt` — day-by-day / stop-by-stop until `trip_complete`; never stop early.

## Probe evidence

shanghai / hangzhou / lisbon: **100%** fill after A+B (see `specs/T5-plan.md` §5.1).

## Non-goals

No day-review LLM; no change to `plan_next_stop` fill implementation; no ADR (prompt hardening only).
