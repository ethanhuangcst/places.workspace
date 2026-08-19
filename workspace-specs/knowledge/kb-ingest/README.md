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

## Indexed 2026-08-19 (earlier batch `8e9c54f5-770f-497a-adef-2476e51aa778`)

| kb title | knowledge_id |
| --- | --- |
| ADR-019: HTTP-first automated user test cases (ChatBox manual deferred) | `3416e508-4886-4b07-94a3-02a821ada703` |
| ADR-020-places-http-only-chat-and-enrich | `e55f155e-fe30-4666-b573-1d7d4d5e7a6c` |
| MCP client integration — Cursor and ChatBox (places-agent) | `9cd60b31-96fc-42f3-a994-8385bd038a9e` |
| places-agent local daemon — make dev vs make up | `919da1b6-8647-4127-8a66-ee3815a575ea` |
| Web app development — HTTP user test automation (TC-H) | `be36e3e7-82a2-4898-9ba7-fb90864cfed6` |
| Places workspace knowledge handbook (refresh 2026-08-19) | `6d924080-46a7-4ec7-b2be-1e6b811bf87f` |
| Architecture — places workspace (refresh 2026-08-19) | `127a8ca4-6987-4d2f-846a-092cc5fa43c9` |

## Indexed 2026-08-19 (confirm round, unique + refresh)

These were proposed then confirmed in chat (operator: ingest and auto confirm). Prefer **refresh** titles over 2026-08-17 copies. Same-title items from the earlier 2026-08-19 batch remain (kb cannot delete); search may return both.

| kb title | knowledge_id | Note |
| --- | --- | --- |
| ADR-011: Independent zh-HK and zh-TW output (no conversion) | `ece56214-c36d-451e-9d25-d0d0c425c775` | New standalone ADR |
| ADR-018: MVP slices by agent capability (amended — MVP-2 includes enrich and chat) | `7f56975d-126e-49a3-aa8c-451361052a1e` | Prefer over older ADR-018 that still says MVP-3 |
| ADR-019: HTTP-first automated user test cases (ChatBox manual deferred) | `4798b686-936f-463c-a931-7bbc03b40b3e` | Duplicate of `3416e508-…` |
| ADR-020: Place chat and Tripadvisor enrich stay HTTP-only | `cd1fa1be-c517-4d52-93db-a84e93a879b3` | Duplicate of `e55f155e-…` |
| MCP client integration — Cursor and ChatBox (places-agent) | `edaf6c06-a021-4d0c-8964-0a74e5149033` | Duplicate of `9cd60b31-…` |
| places-agent local daemon — make dev vs make up | `e3beb8f5-fb67-4ebc-8693-b0acb60e5aef` | Duplicate of `919da1b6-…` |
| places-agent loop and capabilities (refresh 2026-08-19) | `39c20c6c-19bd-47a1-bd66-b8bad39a8ec8` | Prefer over `places-agent-loop` |
| maps-vendor-adapters (refresh 2026-08-19) | `077c73d3-823b-4486-a343-31623d334d31` | Prefer over `maps-vendor-adapters` |
| Operator UI — kb 性冷淡 family (refresh 2026-08-19) | `cd751030-b2e7-4f04-a6dc-c0f1cb8fcef3` | Prefer over 2026-08-17 visual items |
| Web app development — MVP-2 and integration guide addendum | `0bbcbcf0-7c3f-4492-be13-cf96c40e4a1f` | Complements 2026-08-18 slices + TC-H item |
| Places workspace knowledge handbook (refresh 2026-08-19) | `315da401-b1cc-448e-82cc-a399ce7b3dd3` | Duplicate title of `6d924080-…` |
| Architecture — places workspace (refresh 2026-08-19) | `757c280a-6760-4a3b-8400-107159e92015` | Duplicate title of `127a8ca4-…` |

Review pack for the addendum: [`web-app-development-2026-08-19.md`](./web-app-development-2026-08-19.md).

Older kb titles “Places workspace knowledge handbook” and “Architecture — places workspace” remain; prefer the **2026-08-19** refresh copies. kb_organize cannot delete; do not bulk-retag the project.
