---
title: Places workspace knowledge handbook
type: domain-note
status: active
as_of: 2026-08-19
tags:
  - handbook
  - places-agent
  - workspace
related_spec: specs/2.architecture.md
related:
  - knowledge/maps/places-capabilities.md
  - knowledge/maps/vendor-adapters.md
  - knowledge/i18n/hk-tw-output.md
  - knowledge/llm/quanzil-gateway.md
  - knowledge/agent/places-agent-loop.md
  - knowledge/ui/agent-mate-admin-visual.md
  - knowledge/ops/places-agent-next-runtime.md
  - knowledge/web-app-development/lessons-from-places-agent-mvp1.md
  - knowledge/ops/yecao3-places-release.md
  - adr/ADR-001-thin-app-agent-split.md
  - adr/ADR-005-caller-driven-providers.md
  - adr/ADR-009-deploy-option-1.md
  - adr/ADR-010-umbrella-workspace.md
  - adr/ADR-011-hk-tw-independent-locales.md
  - adr/ADR-012-admin-ui-on-agent.md
  - adr/ADR-013-caller-agent-id.md
  - adr/ADR-014-open-meteo-weather.md
  - adr/ADR-015-sqlite-prisma.md
  - adr/ADR-016-custom-http-server.md
  - adr/ADR-017-gmaps-mcp-fallback.md
  - adr/ADR-018-mvp-by-capability.md
  - adr/ADR-019-http-first-user-test-automation.md
  - adr/ADR-020-http-only-chat-and-enrich.md
---

# Places workspace — knowledge handbook

Single consolidated knowledge base for reusable lessons (not binding decisions).  
**Decisions** live in [`../adr/`](../adr/) and [`../2.architecture.md`](../2.architecture.md).  
**Agent AC** lives in [`../1.req-specs.md`](../1.req-specs.md).  
**Provider probe matrix:** [`maps/places-capabilities.md`](./maps/places-capabilities.md).

---

## 1. Spec and workspace split

Umbrella specs are product-family wide; places-agent requirements stay agent-only; architecture holds trust boundaries and deploy. App products get their own req docs later. Parent git tracks specs only; folder is `places-workspace`.

Early `1.req-specs.md` mixed apps, boundaries, geo routing, and agent tools — hard to own and easy to contradict.

| Doc | Put here | Do not put here |
| --- | --- | --- |
| `1.req-specs.md` | places-agent purpose, tools, provider/result contract | App screens, Portainer stacks, browser trust tables |
| `2.architecture.md` | Boundaries, deploy Option 1, workspace layout, ADR index | Per-app AC copy |
| App `*-req-specs.md` (future) | what2eat / where2play UX and product Quanzil | Map vendor adapter details |
| `specs/adr/` | Binding choices among alternatives | Long capability matrices |
| `specs/knowledge/` | Reusable gotchas and ops notes | Binding “we decided X” (use ADR) |

When a new decision appears mid-story: update architecture/ADR first if system shape changes; update agent req only if caller-visible behavior changes.

**Delivery slices** follow agent capabilities (ADR-018), not “admin vs gateway vs intelligence.” All admin UI (Features 14–19) is MVP-1. Canonical table: `1.places-agent/agent-specs/agent-stories.md`.

**Related:** [ADR-010](../adr/ADR-010-umbrella-workspace.md), [ADR-018](../adr/ADR-018-mvp-by-capability.md)

---

## 2. Naming

Machine-readable agent id is **`places-agent`** (host `places.agent-mate.ai`). Display / marketing names for ChatBox etc. may differ; keep the machine id stable.

| Name | Verdict |
| --- | --- |
| `places-agent` | **Accepted** — Places API family; covers eat + play POIs; MCP/API id |
| `place-agent` | Weaker (singular); not used |
| `fun-place-agent` | Rejected — locks “fun”; weak for travel/transit later |
| `play-service` | Early req wording; superseded — confused with where2play brand |
| `geo-capability-route` | Separate-service idea dropped; hard destination router superseded by caller `providers[]` |
| `what2eat-agent` | Rejected as family name — too eat-specific |

Guidance:

- Specs, env `APP_NAME`, MCP `serverInfo.name`, HTTP JSON `agent`, Portainer stack, and agent repo folder all say `places-agent`.
- Callers must see this id (ADR-013). Hostname `places.agent-mate.ai` is not a substitute.
- Parent workspace name is `places-workspace` (not `what2eat.food`).
- Do not revive a standalone `geo-capability-route` deployable or `agent-config/geo-capability-route.json`.

**Related:** [ADR-001](../adr/ADR-001-thin-app-agent-split.md), [ADR-005](../adr/ADR-005-caller-driven-providers.md), [ADR-013](../adr/ADR-013-caller-agent-id.md)

---

## 3. Three “region” axes (do not collapse)

| Axis | Meaning | Who decides |
| --- | --- | --- |
| Search destination | Where the place search is about | Product request (coords / city) — does **not** alone force vendors in the agent |
| Client environment | Where the browser / user is | App UI (e.g. mainland prefers AMAP deeplink when present) |
| Deploy / egress | Where the server runs and which APIs it can reach | Ops / env — keys and network, not product provider policy |

**Avoid:** one JSON that forces `PLACE_SEARCH` by destination and also dictates client map UI and LLM. Those are separate axes.

**Related:** [ADR-005](../adr/ADR-005-caller-driven-providers.md), [ADR-006](../adr/ADR-006-provenance-client-nav.md)

---

## 4. Place providers, provenance, Tripadvisor

Final model: callers pass `providers[]` (and enrich/merge); agent tags `sources[]`; UI picks nav deeplinks by **client** environment.

| Concept | Who decides | Agent role |
| --- | --- | --- |
| Which vendors to call | Caller (`providers[]`, enrich, merge) | Validate credentials / capability; call adapters |
| Result provenance | Agent | Emit `provider` / `sources[]`, optional merge + `primary_provider` |
| Which map app to open | Client / app UI | Return all available secret-free deeplinks |
| LLM model | Deployable env (`OPENAI_*`) | Not tied to destination (§5) |

History: first design used destination buckets (mainland → AMAP; else Google + Tripadvisor) in `agent-config/geo-capability-route.json`. That hard router is **superseded** (ADR-005). The JSON file was **deleted**; supported vendor ids and capability notes live in [`maps/places-capabilities.md`](./maps/places-capabilities.md). Client nav hints belong in the **apps** (ADR-006), not in agent config.

Tripadvisor enrichment:

- HTTP search only: `enrich.tripadvisor` (ADR-020). Not an MCP tool. Not a `providers[]` search vendor.
- Live Terra: `GET /locations/nearby` with `lat`+`lon` (never Google/AMAP ids). Fixture matcher only when mode is not `live`.
- Match by **name + location** (soft signals).
- **Never** pass Google `place_id` (or Google-native ids) to Tripadvisor as a place id.
- Best-effort; failures must not wipe primary search results.

**Config hygiene:** secrets only in env; capability *what each vendor can do* in [`maps/places-capabilities.md`](./maps/places-capabilities.md); adapter gotchas in [`maps/vendor-adapters.md`](./maps/vendor-adapters.md); no agent-editable JSON holding keys.

**Google transport:** direct `maps.googleapis.com` first; Cloudflare Worker MCP (`GMAPS_MCP_*`) only on egress failure. Provenance stays `GOOGLE_MAPS` (ADR-017). Do not treat the Worker as a fourth vendor or as AMAP-when-Google-fails.

**Related:** [ADR-005](../adr/ADR-005-caller-driven-providers.md), [ADR-006](../adr/ADR-006-provenance-client-nav.md), [ADR-007](../adr/ADR-007-tripadvisor-match.md), [ADR-017](../adr/ADR-017-gmaps-mcp-fallback.md), [ADR-020](../adr/ADR-020-http-only-chat-and-enrich.md)

---

## 5. Quanzil / LLM

Product and agent LLM both use **Quanzil** (`OPENAI_*`, `openai` SDK) on the **server** of that deployable. Point `OPENAI_BASE_URL` at Quanzil (`https://quanzil.com/v1`) — **not** `api.openai.com`. LLM is not switched by search destination. Map/place providers are a separate axis (caller-driven). Adapter/gateway notes: [`llm/quanzil-gateway.md`](./llm/quanzil-gateway.md).

| Concern | Config location |
| --- | --- |
| Agent tool-loop LLM | places-agent server env |
| App UX LLM (copy, intent helpers) | that app’s BFF env |
| Which map vendors to hit | request `providers[]` (not LLM env) |

Do not revive `GEO_*_LLM` style destination switches. If a region needs a different model later, treat it as **deploy/tenant config**, not a places destination router.

`sdd.sample/.keys` may list other Quanzil models (e.g. Gemini via Quanzil) for experiments — not destination routing policy. Browser never holds keys.

**Related:** [ADR-004](../adr/ADR-004-quanzil-fixed-per-deployable.md), architecture §5

---

## 6. Trust boundaries (ops reminder)

Four runtimes: consumer browser, operator browser, app BFF, places-agent. Secrets never ship to either browser. Map / Tripadvisor keys only on places-agent (env). Consumer browser → same-origin **app** API → agent (caller key) / product Quanzil. Operator browser → same-origin **places.agent-mate.ai** (admin session) for users and caller keys — never map-vendor keys.

Full table: [`2.architecture.md`](../2.architecture.md) §3. Binding: [ADR-002](../adr/ADR-002-same-origin-bff-trust.md), [ADR-012](../adr/ADR-012-admin-ui-on-agent.md).

Operator chrome is the kb.agent-mate.ai 性冷淡 family (logo on `#fafafa`, locale switcher `EN CN HK TW`). Visual notes: [`ui/agent-mate-admin-visual.md`](./ui/agent-mate-admin-visual.md). Pixel spec: `1.places-agent/agent-specs/3.ui-design.md`.

Local Next 16 gotchas (custom `server.ts`, Edge middleware vs `node:crypto`, Playwright waits): [`ops/places-agent-next-runtime.md`](./ops/places-agent-next-runtime.md).

---

## 7. Deploy on 野草云3 (Option 1)

Ship what2eat, where2play, and places-agent as **three services / three images / prefer three Portainer stacks**. The operator admin UI is **inside** the places-agent image (same hostname), not a fourth stack.

Release-bot operational plan: [`../6.deployment-plan.md`](../6.deployment-plan.md). Places-specific deltas vs kb-agent: [`ops/yecao3-places-release.md`](./ops/yecao3-places-release.md).

| Fact | Value |
| --- | --- |
| Host | 野草云3 / `svr_hk_vps_3` (`38.55.192.140`) |
| Pipeline | GHCR → Portainer → Nginx Proxy Manager → Cloudflare (release-bot pattern) |
| NPM / MCP | places-agent: **one** hostname → **one** container (no kb-style Custom Locations). kb-agent still uses Custom Locations because web and agent are two containers. |
| Apex / domain patterns | mypoke (when needed) |
| Domains | `what2eat.food` (host **`3004`**, not deployed), `where2play.place` (host **`3005`**, not deployed), `places.agent-mate.ai` (**live**, host **`3007` debug**) |
| Thin-app agent URL | `PLACES_AGENT_BASE_URL` + `PLACES_AGENT_CALLER_KEY` (not `PLACES_AGENT_URL` / `PLACES_AGENT_API_KEY`). On-node: `http://places-agent:3000` |
| what2eat LLM / DB | No product `OPENAI_*`. Postgres **`what2eat`** (ADR-023). Child plan: `2.what2eat/2eat-specs/2eat-deployment-plan.md` |
| where2play | Specs + mock-up only. Child plan: `3.where2play/2play-specs/6.deployment-plan.md`. Persistence TBD. |
| Consumer chrome | Shared **places.family** footer row on what2eat / where2play mocks (cross-links + copyright); see what2eat UI guideline §4 |

| Do | Avoid |
| --- | --- |
| One process per image | Multi-process “all apps in one container”; extra admin container |
| Independent stack per **product** | A fourth `places-admin` stack |
| Secrets in Portainer/env | Keys in images, client bundles, or agent-editable JSON |
| Same-origin BFF on each web surface | Browser → vendor keys; admin session used as a caller API key |

places-agent stack also needs: Resend + session secret in env; PostgreSQL `places_agent` (ADR-025); process entry `server.ts` (ADR-016); default admin seed on first boot.

Adding a **fourth consumer product**: default to another image + stack. Do not treat the operator UI as that fourth product.

**Related:** [ADR-009](../adr/ADR-009-deploy-option-1.md), [ADR-012](../adr/ADR-012-admin-ui-on-agent.md), [ADR-015](../adr/ADR-015-sqlite-prisma.md) (superseded), [ADR-025](../adr/ADR-025-places-agent-postgres-prisma.md), [ADR-016](../adr/ADR-016-custom-http-server.md)

---

## 8. Workspace layout (reminder)

Parent: `~/code/places-workspace/` — umbrella **specs** only. Children are separate remotes, gitignored from parent: `0.1.sdd.sample`, `0.2.release-bot`, `1.places-agent`, `2.what2eat`, `3.where2play`.

`0.1.sdd.sample` origin is **https://github.com/ethanhuangcst/sdd.sample.git** (not `sdd-example.git`). Local `.keys` in that folder is a gitignored inventory, not part of the sample repo.

**Related:** [ADR-010](../adr/ADR-010-umbrella-workspace.md), architecture §11

---

## 9. Locales (HK vs TW)

`HK` and `TW` are independent Traditional Chinese locales. Do not OpenCC-convert catalogs. Three layers: agent copy (catalogs), vendor facts (`languageCode`), formatters (`Intl` + real currency). LLM narrative loads a small glossary on demand. **Weather is Open-Meteo only** (ADR-014): WMO codes are Layer A (`weather.wmo.{code}`), not Layer B pass-through — Open-Meteo responds in English/numeric codes and must be translated. Full table and anti-patterns: [`i18n/hk-tw-output.md`](./i18n/hk-tw-output.md). Decisions: [ADR-011](../adr/ADR-011-hk-tw-independent-locales.md), [ADR-014](../adr/ADR-014-open-meteo-weather.md).

Agent loop and capability list: [`agent/places-agent-loop.md`](./agent/places-agent-loop.md).

---

## ADR quick map

| ID | Topic |
| --- | --- |
| ADR-001 | Thin-app / agent split |
| ADR-002 | Same-origin BFF trust |
| ADR-003 | Dual transport (HTTP + MCP) |
| ADR-004 | Quanzil fixed per deployable |
| ADR-005 | Caller-driven providers |
| ADR-006 | Provenance + client nav |
| ADR-007 | Tripadvisor match without ID passthrough |
| ADR-008 | Itinerary engine vs trip UX |
| ADR-009 | Deploy Option 1 |
| ADR-010 | Umbrella workspace |
| ADR-011 | Independent HK/TW locales |
| ADR-012 | Admin UI on the places-agent deployable |
| ADR-013 | Caller-visible agent id `places-agent` |
| ADR-014 | Open-Meteo weather; localize English/WMO output |
| ADR-015 | SQLite + Prisma on the places-agent volume (**superseded** by ADR-025) |
| ADR-016 | Custom Node HTTP server as process entry |
| ADR-017 | Google Maps Worker MCP as Google transport fallback |
| ADR-018 | MVP slices by agent capability; all admin UI in MVP-1 |
| ADR-019 | HTTP-first user tests (TC-H in CI; ChatBox TC-C deferred) |
| ADR-020 | Place chat and Tripadvisor enrich stay HTTP-only |
| ADR-021 | Live vendor mode must not serve fixture data |
| ADR-022 | Timed itinerary (`detail: timed`) |
| ADR-023 | what2eat Postgres + Prisma |
| ADR-024 | Quality gates on TypeScript 7 (Babel ESLint, coverage, isolated E2E) |
| ADR-025 | places-agent PostgreSQL + Prisma (supersedes ADR-015) |
