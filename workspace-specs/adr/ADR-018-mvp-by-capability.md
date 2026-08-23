# ADR-018: MVP slices by agent capability; all admin UI in MVP-1

## Status
Accepted

## Context

places-agent has 19 backlog features: traveler tools (1–10), HTTP/MCP and caller keys (11–12), locales (13), and operator admin UI (14–19). An earlier slice plan grouped work as **operator host** (admin + channels, no place tools) → **place gateway** (all search/details/nav) → **agent intelligence** (Tripadvisor, itinerary, NL chat).

That grouping hid **what the agent can do**. The operator then required two constraints: plan MVPs **by agent capabilities**, and keep **all admin UI** in MVP-1.

## Decision

1. Slice delivery by **caller-facing capability**, not by “admin vs gateway vs LLM.”
2. **MVP-1** includes **every admin UI feature (14–19)** plus Call (11–12) plus **search restaurants** and the plumbing that makes a restaurant card usable: details, geocode, navigate, `providers[]`, `sources[]`, tool locales (1, 3–7, 13). No OPENAI_CN chat loop in this slice.
3. **MVP-2** is **search places** (2), **plan itinerary** (9, including Open-Meteo `weather.wmo.*` keys), **Tripadvisor enrich** (8), and **NL place chat** (10). Chat is a loop over tools already shipped. (Amended 2026-08-18: former MVP-2 and MVP-3 are one slice.)
4. Do not start MVP-2 while any of 14–19 is unfinished.

Canonical table: [`1.places-agent/agent-specs/agent-stories.md`](../../1.places-agent/agent-specs/agent-stories.md). Build order: [`agent-design.md`](../../1.places-agent/agent-specs/agent-design.md) §16.

## Rationale

- Agent-builder guidance: start with a few capabilities and add when the model (or caller) fails for lack of one — not when an internal layer is “complete.”
- Admin-only MVP-1 would ship a host with nothing a BFF can search. Restaurant search is the first what2eat capability. Place search, itinerary, Tripadvisor enrich, and chat ship together as MVP-2 so where2play and ChatBox land on one remaining slice; chat is last inside that slice because it reuses the tool core.
- Splitting admin screens across MVPs would leave operators unable to invite, issue keys, or switch locale while restaurant search landed — rejected.
- Putting all discovery tools (restaurants **and** places) in MVP-1 would mix two capabilities and delay itinerary for where2play.

Rejected: admin-only first slice; splitting Features 14–19; NL chat as the only way to search; a second tool core for MCP vs HTTP.

## Consequences

- MVP-1 is larger than “sign-in and keys” but is usable: operators manage the host; what2eat can call `search_restaurants` on HTTP and MCP.
- Feature 13 tool/card catalogs ship in MVP-1; weather label keys wait for Feature 9 (MVP-2).
- Incremental delivery still applies: one user story to DoD inside the slice. Suggested MVP-1 order is in the stories MVP plan.

## Date
2026-08-17 (amended 2026-08-18: merge former MVP-3 into MVP-2)
