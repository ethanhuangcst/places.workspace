# ADR-076: MCP public surface — `plan_trip` + `fetch_trip_details` only

Family backlog: [`product-backlog.md`](../product-backlog.md) · MVP-T11 docs closeout

## Status

**Accepted**（2026-09-22）· **Implemented**（2026-09-22）— MCP `create-server.ts` registers only `plan_trip` + `fetch_trip_details`

## Context

[agent-design](../agent-specs/agent-design.md) host contract and [ADR-050](./ADR-050-where2play-no-product-llm.md) already state that Cursor / ChatBox hosts should call **two** public methods: `plan_trip` and `fetch_trip_details`. Internal intents (intake, discover, make, fill, tips, visa write) run inside the agent loop and must not be separate MCP tools for new product paths.

[ADR-040](./ADR-040-plan-itinerary-align-split-tools.md) D5/D7 and [ADR-043](./ADR-043-chatbox-mcp-and-cross-product-closure.md) still describe a larger MCP list (`arrange_day`, `plan_itinerary` aliases, discover → make → plan_next_stop). That list is **as-built and superseded as the target public surface** by this ADR. Those ADR texts are not rewritten; this document is the current decision for the public MCP tool list.

As of 2026-09-22, MCP registration was reduced to the two public tools (see Implementation). HTTP `/v1` routes for search/geocode/visa/discover/make/fill are unchanged.

## Decision

### D1 — MCP target public tools

MCP `tools/list` **target** contains only:

| Tool | Role |
| --- | --- |
| `plan_trip` | Write / intake / full loop (lazy `trip_id`) |
| `fetch_trip_details` | Read partitions by `fields[]` |

### D2 — Stay on HTTP for BFFs (not MCP product path)

These remain HTTP `/v1` for where2play / what2eat BFF. **Not** MCP product tools:

- `search_restaurants`, `search_places`, `suggest_places`, `geocode`, `visa_requirement`
- HTTP `plan_trip` / `fetch_trip_details` (same handlers as MCP)

### D3 — Legacy: not for new MCP product scheduling

Do not schedule new ChatBox / Cursor flows through:

- `arrange_day`
- `discover_places`, `make_itinerary`, `plan_next_stop`
- `travel_tips` (tips live under `plan_trip` / `artifacts` + fetch)
- Aliases `plan_itinerary`, `trip_plan`, `trips`

HTTP may keep those routes until a separate cleanup story.

### D4 — Implementation

[`places-agent/src/mcp/create-server.ts`](../../places-agent/src/mcp/create-server.ts) registers **only** `plan_trip` and `fetch_trip_details`. MCP contract tests assert this list.

### D5 — Relation to prior ADRs

- Supersedes **as the public MCP tool catalog** the lists implied by ADR-040 D5/D7 and ADR-043 MCP tables for *new* host integrations.
- Does not reopen ADR-071 (refine cancelled) or ADR-050 (2play zero product LLM).
- Does not grow city encyclopedias (ADR-042).

## Rationale

- One loop, two host methods, matches agent-design and reduces host misfires across multi-MCP sessions.
- BFFs still need thin HTTP search/geocode/visa; those are not Cursor natural-language tools.

## Consequences

- Cursor / Claude `tools/list` shows two tools.
- HTTP BFF routes unchanged.
- Guide / ChatBox ops copy should prefer `plan_trip` + `fetch_trip_details` only.

## Date

2026-09-22
