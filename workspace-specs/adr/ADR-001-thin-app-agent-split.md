# ADR-001: Thin-app / agent split

## Status

Accepted

## Decision

what2eat and where2play are thin web + BFF products. places-agent owns place/map tools, vendor adapters, agent LLM tool loops, and `plan_itinerary`.

**where2play LLM:** planning and in-page chat use places-agent Quanzil only — the where2play app does **not** hold product `OPENAI_*` (see ADR-033, `2play-design.md` §2.1). what2eat may still use app-owned Quanzil for product-owned paths.

## Consequences

App repos stay UX-focused; place-provider complexity and keys centralize on places-agent. where2play secrets inventory excludes product LLM keys.
