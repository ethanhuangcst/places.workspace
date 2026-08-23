# places-agent arrange P0 (nullish date + slim candidates)

**Date:** 2026-08-22  
**Related:** [`performance.md`](../../../1.places-agent/agent-specs/performance.md) §4–§5, story `places-agent-itinerary-mcp-p0`

## Lessons

1. **JSON `date: null` ≠ omitted.** Zod `z.string().optional()` rejects `null` (MCP hosts often send null). Use `z.string().nullish()` on HTTP `arrangeDayBody` and MCP `arrange_day`.
2. **Slim before LLM, join photos after.** Strip `photos` / `hours` / `sources` / deeplinks for prompt assembly (`slimArrangeCandidate`); keep the original candidate pool for Phase 4 photo attach by name.
3. **MCP descriptions are a routing control.** Mutual exclusion (`discover`/`arrange` vs `plan_itinerary`) and “do not echo photos/hours” must appear in tool `description` strings — hosts follow them more reliably than separate docs.

## Tests

- `src/http/schemas.test.ts` — `date: null`
- `src/core/itinerary-planner.test.ts` — slim + Phase 4 photos
- `tests/mcp.test.ts` — description keywords + `date: null` call
