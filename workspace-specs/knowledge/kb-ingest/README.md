# KB ingest pack (places-workspace, 2026-08-17)

Self-contained lessons for **kb-agent**. No secrets. Product AC stays in agent-specs; this pack is reusable facts and decisions.

kb writes are **propose → human confirm**. This folder is the review copy. Do not re-ingest items already in the library.

## Already in kb (skip)

| Title in kb | Local source |
| --- | --- |
| ADR-001 … ADR-010 | `workspace-specs/adr/` |
| ADR-014, 015, 016, 017 | same |
| Places workspace knowledge handbook (older snapshot; missing admin / ADR-011+) | `knowledge/handbook.md` |
| Architecture — places workspace (older; three runtimes, ADR index stops at 010) | `2.architecture.md` |
| places-agent — requirements | `1.req-specs.md` |
| Places API capability matrix | `knowledge/maps/places-capabilities.md` |
| maps-vendor-adapters | `knowledge/maps/vendor-adapters.md` |
| Quanzil gateway | `knowledge/llm/quanzil-gateway.md` |
| HK vs TW three layers + weather Layer A | `knowledge/i18n/hk-tw-output.md` |
| **Web app development —** (7 items, 2026-08-18) | `knowledge/web-app-development/` — see kb-ingest README for IDs |
| geo-capability-route (superseded) | do not revive |

## Gaps proposed (2026-08-17)

Indexed 2026-08-17 (batch `d3849e74-d898-4b30-b1dd-6d52a53a611a`).

| Filename | Why ingest |
| --- | --- |
| `adr-012-admin-ui.md` | Operator UI on the agent host; session ≠ caller key; no fourth stack |
| `adr-013-agent-id.md` | Machine id `places-agent` vs hostname; MCP `serverInfo.name` / HTTP `agent` |
| `adr-018-mvp-capability.md` | Capability slices; all admin UI in MVP-1 |
| `places-agent-loop.md` | Tool loop, 5 capabilities, no workflow engine, no extra MLOps |
| `agent-mate-admin-visual.md` | kb 性冷淡 family for other `*.agent-mate.ai` operator UIs |
| `places-agent-next-runtime.md` | Next 16 custom server, Edge vs `node:crypto`, Playwright, Prisma env |

## Do not ingest

- Story order `14 → 15 → …` (lives in `1.agent-stories.md`; will rot)
- `.env.local` / `.keys` values
- Clickable mock-up HTML
- Duplicate handbook / architecture dumps (prefer these deltas until a full handbook refresh is decided)

## Stale in kb (optional later refresh)

Handbook and architecture copies in kb still describe **three** runtimes (no operator browser) and ADR index through **010**. After these six land, a handbook refresh is optional — do not paste the whole file on top without replacing the old item.

## Refreshed 2026-08-17

Indexed (batch `998b3471-3e95-4bca-9605-169510140523`):

- `handbook-2026-08-17.md` — four runtimes, ADR-001–018
- `architecture-2026-08-17.md` — operator UI on agent, ADR index through 018

## Gaps proposed (2026-08-18) — category web-app-development

Source: [`../web-app-development/lessons-from-places-agent-mvp1.md`](../web-app-development/lessons-from-places-agent-mvp1.md). Review pack: [`web-app-development-2026-08-18.md`](./web-app-development-2026-08-18.md).

**Indexed 2026-08-18** (all confirmed via kb-agent):

| kb title | knowledge_id |
| --- | --- |
| Web app development — auth forms and token-in-email flows | `ec4222a7-ef3a-440e-a9ee-630a17b5f53a` |
| Web app development — Next.js 16 admin app constraints | `dc04bc12-6f0a-45d8-9af8-9590345df1e4` |
| Web app development — cross-machine local dev | `7fba440c-a032-4da0-942c-629a8ca1ca90` |
| Web app development — Playwright operator E2E | `4f643530-dc1b-4e9b-876f-64d4e5cdaf90` |
| Web app development — admin mail and destructive mutations | `30974b57-60ef-4baa-ac76-4824389a2295` |
| Web app development — i18n and dual auth testing | `d3ba04fc-77a3-493e-ae9c-8b4a6de9776b` |
| Web app development — DoD gates for admin slices | `06675c3b-3662-4604-a1d8-0a65825e4a39` |

If kb already has “places-agent Next.js 16 local runtime” from the 2026-08-17 pack, **merge or supersede** with the Next.js row above — do not duplicate.

Older kb titles “Places workspace knowledge handbook” and “Architecture — places workspace” remain; prefer the refresh copies (they say so in the first paragraph). kb_organize cannot delete; do not bulk-retag the project.
