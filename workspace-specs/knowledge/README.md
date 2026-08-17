# Knowledge Base

Reusable research conclusions, ops lessons, and domain notes (not code truth).

| Layer | Location |
| --- | --- |
| Product requirements | [`../1.req-specs.md`](../1.req-specs.md) (+ future app req docs) |
| Architecture decisions | [`../2.architecture.md`](../2.architecture.md), [`../adr/`](../adr/) |
| Knowledge (this tree) | Consolidated handbook + probe matrix |

## Canonical docs

| Doc | Topic | Updated |
| --- | --- | --- |
| [handbook.md](./handbook.md) | **All** consolidated knowledge (naming, axes, providers, LLM, deploy, workspace) | 2026-08-17 |
| [maps/places-capabilities.md](./maps/places-capabilities.md) | AMAP / Google / Tripadvisor capability probe matrix | 2026-08-14 |
| [maps/vendor-adapters.md](./maps/vendor-adapters.md) | AMAP lng,lat/GCJ-02; Google direct+MCP; Open-Meteo WMO i18n | 2026-08-17 |
| [llm/quanzil-gateway.md](./llm/quanzil-gateway.md) | Quanzil via `openai` SDK; not `api.openai.com` | 2026-08-17 |
| [i18n/hk-tw-output.md](./i18n/hk-tw-output.md) | HK vs TW three-layer output, glossary, Google `languageCode`, Open-Meteo weather codes | 2026-08-17 |
| [agent/places-agent-loop.md](./agent/places-agent-loop.md) | Tool loop, 5 capabilities, prompt versioning (MVP vs later) | 2026-08-17 |

Older topic files under `architecture/`, `llm/`, `maps/provider-selection-*`, `naming/`, `ops/` were merged into `handbook.md` and removed.
