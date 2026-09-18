# plan_trip refine mode (MVP-T9 / agent-chat-93e)

**Date:** 2026-09-18

## Decision

Chat 改行程 uses **`plan_trip` refine mode** (`refine.instruction` + required `trip_id`), not a separate tool or full-loop rebuild. Model loop tools: `search_places`, `commit_trip` (`operations[]`), `stop`.

## Patch schema

`commit_trip.operations[]`:

| op | fields |
| --- | --- |
| `remove_stop` | `day_index` (1-based), `stop_index` (0-based in day.stops) |
| `replace_stop` | `day_index`, `stop_index`, `name` (must ground via search or candidates) |
| `swap_stops` | `day_index`, `from_index`, `to_index` |

`artifacts.filled_stops` reconciled by day + stop name after patch. No city encyclopedia in source (ADR-042).

## Tests

`places-agent/src/core/plan-trip-refine.test.ts` — fixture Trip + `_testRefineTurns`.

## Related

ADR-050 (single brain), ADR-046 (Trip Store revision).
