# Knowledge Base

Reusable research conclusions, ops lessons, and domain notes (not code truth).

| Layer | Location |
| --- | --- |
| Product requirements | [`../1.req-specs.md`](../1.req-specs.md) (+ future app req docs) |
| Architecture decisions | [`../2.architecture.md`](../2.architecture.md), [`../adr/`](../adr/) |
| Deployment (release-bot) | [`../6.deployment-plan.md`](../6.deployment-plan.md) |
| Knowledge (this tree) | Consolidated handbook + probe matrix |

## Canonical docs

| Doc | Topic | Updated |
| --- | --- | --- |
| [handbook.md](./handbook.md) | **All** consolidated knowledge (naming, axes, providers, LLM, deploy, workspace) | 2026-08-17 |
| [maps/places-capabilities.md](./maps/places-capabilities.md) | AMAP / Google / Tripadvisor capability probe matrix | 2026-08-14 |
| [maps/vendor-adapters.md](./maps/vendor-adapters.md) | AMAP lng,lat/GCJ-02; Google direct+MCP; Open-Meteo WMO i18n | 2026-08-17 |
| [llm/quanzil-gateway.md](./llm/quanzil-gateway.md) | Quanzil via `openai` SDK; not `api.openai.com` | 2026-08-17 |
| [i18n/hk-tw-output.md](./i18n/hk-tw-output.md) | HK vs TW three-layer output, glossary, Google `languageCode`, Open-Meteo weather codes | 2026-08-17 |
| [agent/places-agent-loop.md](./agent/places-agent-loop.md) | Tool loop, 5 capabilities, ADR-018 delivery slices | 2026-08-17 |
| [ui/agent-mate-admin-visual.md](./ui/agent-mate-admin-visual.md) | kb 性冷淡 operator chrome; logo on `#fafafa`; four locale ids | 2026-08-17 |
| [ops/places-agent-next-runtime.md](./ops/places-agent-next-runtime.md) | Next 16 custom server, Edge middleware, Playwright waits, Prisma env | 2026-08-17 |
| [kb-ingest/README.md](./kb-ingest/README.md) | What is already in kb vs propose-only gaps | 2026-08-17 |
| [ops/yecao3-places-release.md](./ops/yecao3-places-release.md) | 野草云3 deltas vs kb-agent (one container, SQLite, proposed ports) | 2026-08-17 |
| [ops/places-agent-admin-invite-dev.md](./ops/places-agent-admin-invite-dev.md) | Cross-machine invite testing; LAN dev origins; POST-not-GET gate | 2026-08-18 |
| [web-app-development/README.md](./web-app-development/README.md) | **Consolidated** Next/auth/E2E/mail lessons from MVP-1; KB ingest manifest | 2026-08-18 |
| [web-app-development/lessons-from-places-agent-mvp1.md](./web-app-development/lessons-from-places-agent-mvp1.md) | Full consolidated body (auth, Next 16, Playwright, cross-machine, DoD) | 2026-08-18 |

Older topic files under `architecture/`, `llm/`, `maps/provider-selection-*`, `naming/`, `ops/` were merged into `handbook.md` and removed.
