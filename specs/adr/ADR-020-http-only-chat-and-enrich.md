# ADR-020: Place chat and Tripadvisor enrich stay HTTP-only

## Status
Accepted

## Context

[ADR-003](./ADR-003-dual-transport.md) exposes the same places-agent **tools** on HTTP and MCP over one core. A naive reading is: every caller-facing capability, including NL chat and Tripadvisor ratings, must appear as an MCP tool.

MVP-2 shipped six public tools plus two extra HTTP surfaces:

- `POST /v1/chat` — OPENAI_CN tool loop for app BFFs
- `enrich.tripadvisor` on HTTP search — best-effort ratings/content (ADR-007)

MCP hosts (Cursor, ChatBox) already **are** the agent loop. Tripadvisor enrich is a server-side flag, not a seventh public tool and not a `providers[]` search vendor.

## Decision

1. **Dual transport (ADR-003) applies to the six tools only:** `search_restaurants`, `search_places`, `get_place_details`, `geocode`, `navigate`, `plan_itinerary`.
2. **Place chat is HTTP-only** — `POST /v1/chat`. Do not register it as an MCP tool.
3. **Tripadvisor enrich is HTTP-only** — request body `enrich.tripadvisor` on search. Omit it from MCP tool schemas. Do not accept `TRIPADVISOR` in `providers[]` for search/details (capability_unsupported). Open-Meteo stays a private itinerary helper, not a public tool.

This does **not** supersede ADR-003. It states the exceptions.

## Rationale

- Wrapping chat as an MCP tool nests agent-in-agent: the host would call places-agent chat, which would call tools the host can already call.
- Enrich is name+location matching (ADR-007), not a model-facing capability. Callers that need ratings use HTTP search; MCP callers use the six tools and skip enrich.
- Rejected: seventh public tool; `TRIPADVISOR` in `providers[]`; chat on `/mcp`.

## Consequences

- Integration guide Channel column: six tools **HTTP and MCP**; Place chat **HTTP only** + `POST /v1/chat`; `Tripadvisor.enrich` **HTTP only**.
- Guide Capabilities cell for enrich uses the display literal `Tripadvisor.enrich` (dot). The HTTP JSON path remains `enrich.tripadvisor`.
- Contract tests: MCP schemas have six tools and no `enrich`; HTTP search accepts the enrich flag.
- ChatBox / Cursor docs stay on `/mcp` + the six tools, not `/v1/chat`.

## Date
2026-08-19
