# ADR-001: Thin-app / agent split

## Status

Accepted

## Decision

what2eat and where2play are thin web + BFF products. places-agent owns place/map tools, vendor adapters, agent LLM tool loops, and `plan_itinerary`.

## Consequences

App repos stay UX-focused; place-provider complexity and keys centralize on places-agent.
