---
title: places-agent loop and capability design
type: design-direction
status: active
as_of: 2026-08-17
tags:
  - agent
  - quanzil
  - mcp
related_spec: workspace-specs/2.architecture.md
related:
  - knowledge/handbook.md
  - knowledge/i18n/hk-tw-output.md
  - adr/ADR-001-thin-app-agent-split.md
  - adr/ADR-003-dual-transport.md
  - adr/ADR-004-quanzil-fixed-per-deployable.md
  - adr/ADR-011-hk-tw-independent-locales.md
  - adr/ADR-014-open-meteo-weather.md
---

# places-agent — loop, capabilities, and MLOps (MVP)

## Summary

places-agent is a **tool loop** with a **fixed Quanzil per deployable**, not a training platform. Keep the loop boring; add locale and provenance as request/response contracts. Defer feature stores, Kubeflow, and multi-model routing.

## Evidence

- Architecture: HTTP + MCP, one tool core (ADR-003); LLM not destination-routed (ADR-004); thin apps (ADR-001).
- Agent-builder: model already knows how to be an agent; code supplies capabilities + guardrails.
- Constraints: small team, 野草云3 / Option 1, map vendor keys on the server, caller API keys for HTTP/MCP.

## Lesson / guidance

### Loop (do not invent a workflow engine)

```
LOOP:
  Model sees: messages + locale + available tools
  Model decides: tool call or final answer
  If tool: execute adapter (AMAP / Google / Tripadvisor) or Open-Meteo weather, append result (truncated), continue
  If answer: apply Layer A/C catalogs (including `weather.wmo.{code}`) + Layer L glossary; return
```

Trust the model to sequence search → details → navigate. Do **not** hard-code “always call geocode then search”.

### Capabilities (start with these; add only when the model fails)

| Tool | Why it exists |
| --- | --- |
| `search_restaurants` / `search_places` | Discovery |
| `place_details` | Card + hours + provenance |
| `geocode` | NL place → lat/lng |
| `navigate` | Deep links + optional Directions/Routes |
| `plan_itinerary` | Multi-stop engine (Feature 9) |

Tripadvisor enrich is a **server-side enrichment** on details/search, not a sixth tool the model must remember unless match quality needs an explicit call.

**Delivery (capability slices):** MVP-1 = all admin UI + HTTP/MCP + `search_restaurants` (plus details, geocode, navigate, vendors, sources, locales). MVP-2 = `search_places` + `plan_itinerary`. MVP-3 = Tripadvisor enrich + NL chat loop. See `1.places-agent/agent-specs/1.agent-stories.md`.

Knowledge (HK/TW glossary, itinerary pacing rules): **load when `locale` or plan mode requires it**, not in every system prompt.

### Context hygiene

- Truncate vendor JSON (keep id, name, location, rating, provenance).
- Isolate long itinerary planning if it pollutes chat (Level 2 progress list; subagents only if exploration dumps kill the thread).
- File/image upload: extract text/geo hints into a short user message; do not keep raw bytes in the loop.

### AI architecture — tiered

| Now (MVP) | Later (only if measured need) |
| --- | --- |
| One Quanzil + versioned system prompt + glossary file | Prompt registry / A/B |
| Adapter latency + vendor error logs | Quality eval set (HK vs TW golden strings) |
| Cache geocode/place_id lookups | Online feature store |
| Caller API key auth | Per-key quotas, canary model |

**Version** from day one: prompt id, glossary id, catalog locale pack. Rollback = pin those ids (no GPU serving stack).

### Anti-patterns for this repo

- Mock-only map tools marked “done”
- One Traditional Chinese catalog for HK and TW
- Geo-forced AMAP vs caller `providers[]` (ADR-005)
- Front-loading Wikipedia-scale glossaries every turn
- Building Kubeflow/MLflow for a single FastAPI (or equivalent) agent

## Links

- [ADR-011](../../adr/ADR-011-hk-tw-independent-locales.md)
- [HK/TW output](../i18n/hk-tw-output.md)
