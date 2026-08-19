---
title: places-agent loop and capability design
type: design-direction
status: active
as_of: 2026-08-19
tags:
  - agent
  - quanzil
  - mcp
related_spec: workspace-specs/2.architecture.md
related:
  - knowledge/handbook.md
  - knowledge/i18n/hk-tw-output.md
  - knowledge/maps/vendor-adapters.md
  - adr/ADR-001-thin-app-agent-split.md
  - adr/ADR-003-dual-transport.md
  - adr/ADR-004-quanzil-fixed-per-deployable.md
  - adr/ADR-007-tripadvisor-match.md
  - adr/ADR-011-hk-tw-independent-locales.md
  - adr/ADR-014-open-meteo-weather.md
  - adr/ADR-018-mvp-by-capability.md
  - adr/ADR-020-http-only-chat-and-enrich.md
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

Six public tools on **HTTP and MCP** (ADR-003). Two extra HTTP surfaces are **not** MCP tools (ADR-020).

| Capability | Channel | Why it exists |
| --- | --- | --- |
| `search_restaurants` / `search_places` | HTTP and MCP | Discovery |
| `get_place_details` | HTTP and MCP | Card + hours + provenance |
| `geocode` | HTTP and MCP | NL place → lat/lng |
| `navigate` | HTTP and MCP | Deep links + optional Directions/Routes |
| `plan_itinerary` | HTTP and MCP | Multi-stop engine (Feature 9) |
| Place chat (`POST /v1/chat`) | HTTP only | BFF NL loop over the same six tools. MCP hosts already are the loop — do not nest chat as an MCP tool. |
| `Tripadvisor.enrich` (`enrich.tripadvisor` on HTTP search) | HTTP only | Server-side ratings/content (ADR-007). Not a `providers[]` vendor. Omit from MCP schemas. |

Guide copy: first six Capabilities cells are **tool-name literals**. Last two are i18n **Place chat** and the display literal **`Tripadvisor.enrich`** (dot) — not the JSON path `enrich.tripadvisor`. Open-Meteo stays a private itinerary helper (`weather.wmo.*`), not a public tool.

**Delivery (capability slices, ADR-018):** do not slice as “operator host → place gateway → intelligence.” MVP-1 = **all** admin UI + HTTP/MCP + `search_restaurants` (plus details, geocode, navigate, vendors, sources, locales) so what2eat can call a real tool. MVP-2 = `search_places` + `plan_itinerary` + Tripadvisor enrich + NL chat loop over the same tool core. Feature 13 weather keys wait for itinerary. Canonical table: `1.places-agent/agent-specs/1.agent-stories.md`.

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
- Exposing `POST /v1/chat` or Tripadvisor enrich as MCP tools (ADR-020)
- Front-loading Wikipedia-scale glossaries every turn
- Building Kubeflow/MLflow for a single FastAPI (or equivalent) agent

## Links

- [ADR-011](../../adr/ADR-011-hk-tw-independent-locales.md)
- [ADR-018](../../adr/ADR-018-mvp-by-capability.md)
- [ADR-020](../../adr/ADR-020-http-only-chat-and-enrich.md)
- [HK/TW output](../i18n/hk-tw-output.md)
