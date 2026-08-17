---
title: Places workspace knowledge handbook
type: domain-note
status: active
as_of: 2026-08-17
tags:
  - handbook
  - places-agent
  - workspace
related_spec: specs/2.architecture.md
related:
  - knowledge/maps/places-capabilities.md
  - adr/ADR-001-thin-app-agent-split.md
  - adr/ADR-005-caller-driven-providers.md
  - adr/ADR-009-deploy-option-1.md
  - adr/ADR-010-umbrella-workspace.md
---

# Places workspace — knowledge handbook

Single consolidated knowledge base for reusable lessons (not binding decisions).  
**Decisions** live in [`../adr/`](../adr/) and [`../2.architecture.md`](../2.architecture.md).  
**Agent AC** lives in [`../1.req-specs.md`](../1.req-specs.md).  
**Provider probe matrix:** [`maps/places-capabilities.md`](./maps/places-capabilities.md).

---

## 1. Spec and workspace split

Umbrella specs are product-family wide; places-agent requirements stay agent-only; architecture holds trust boundaries and deploy. App products get their own req docs later. Parent git tracks specs only after rename to `places-workspace`.

Early `1.req-specs.md` mixed apps, boundaries, geo routing, and agent tools — hard to own and easy to contradict.

| Doc | Put here | Do not put here |
| --- | --- | --- |
| `1.req-specs.md` | places-agent purpose, tools, provider/result contract | App screens, Portainer stacks, browser trust tables |
| `2.architecture.md` | Boundaries, deploy Option 1, workspace layout, ADR index | Per-app AC copy |
| App `*-req-specs.md` (future) | what2eat / where2play UX and product Quanzil | Map vendor adapter details |
| `specs/adr/` | Binding choices among alternatives | Long capability matrices |
| `specs/knowledge/` | Reusable gotchas and ops notes | Binding “we decided X” (use ADR) |

When a new decision appears mid-story: update architecture/ADR first if system shape changes; update agent req only if caller-visible behavior changes.

**Related:** [ADR-010](../adr/ADR-010-umbrella-workspace.md)

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

- Specs, env `APP_NAME`, MCP server id, and agent repo folder all say `places-agent`.
- Parent workspace name is `places-workspace` (folder rename after architecture), not `what2eat.food`.
- Do not revive a standalone `geo-capability-route` deployable.

**Related:** [ADR-001](../adr/ADR-001-thin-app-agent-split.md), [ADR-005](../adr/ADR-005-caller-driven-providers.md)

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

History: first design used destination buckets (mainland → AMAP; else Google + Tripadvisor) in `agent-config/geo-capability-route.json`. That hard router is **superseded**; the file now lists supported provider ids and optional client nav hints only.

Tripadvisor enrichment:

- Match by **name + location** (soft signals).
- **Never** pass Google `place_id` (or Google-native ids) to Tripadvisor as a place id.
- Best-effort; failures must not wipe primary search results.

**Config hygiene:** secrets only in env; capability *what each vendor can do* in [`maps/places-capabilities.md`](./maps/places-capabilities.md); no agent-editable JSON holding keys.

**Related:** [ADR-005](../adr/ADR-005-caller-driven-providers.md), [ADR-006](../adr/ADR-006-provenance-client-nav.md), [ADR-007](../adr/ADR-007-tripadvisor-match.md)

---

## 5. Quanzil / LLM

Product and agent LLM both use OpenAI Quanzil (`OPENAI_*`) on the **server** of that deployable. LLM is not switched by search destination. Map/place providers are a separate axis (caller-driven).

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

Three runtimes: browser, app BFF, places-agent. Secrets never ship to the browser. Map / Tripadvisor keys only on places-agent. Browser → same-origin app API → agent / Quanzil.

Full table: [`2.architecture.md`](../2.architecture.md) §3. Binding decision: [ADR-002](../adr/ADR-002-same-origin-bff-trust.md).

---

## 7. Deploy on 野草云3 (Option 1)

Ship what2eat, where2play, and places-agent as **three services / three images / prefer three Portainer stacks**.

| Fact | Value |
| --- | --- |
| Host | 野草云3 / `svr_hk_vps_3` (`38.55.192.140`) |
| Pipeline | GHCR → Portainer → Nginx Proxy Manager → Cloudflare (release-bot pattern) |
| MCP Custom Locations template | kb-agent |
| Apex / domain patterns | mypoke (when needed) |
| Domains | `what2eat.food`, `where2play.place`, `places.agent-mate.ai` |

| Do | Avoid |
| --- | --- |
| One process per image | Multi-process “all apps in one container” |
| Independent stack per product when blast radius matters | Shared stack that couples rollbacks |
| Secrets in Portainer/env | Keys in images, client bundles, or agent-editable JSON |
| Same-origin BFF on each web app | Browser → places-agent with vendor keys |

Adding a fourth product: default to another image + stack.

**Related:** [ADR-009](../adr/ADR-009-deploy-option-1.md)

---

## 8. Workspace layout (reminder)

Parent (soon): `~/code/places-workspace/` — umbrella **specs** only. Children are separate remotes, gitignored from parent: `release-bot`, `sdd.sample`, `places-agent`, `what2eat`, `where2play`.

Current folder name `what2eat.food` is transitional until rename.

**Related:** [ADR-010](../adr/ADR-010-umbrella-workspace.md), architecture §11

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
