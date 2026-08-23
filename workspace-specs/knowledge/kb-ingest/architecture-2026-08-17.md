# Architecture — places workspace (refresh 2026-08-17)

Supersedes the earlier kb item titled “Architecture — places workspace” that described three runtimes and an ADR index stopping at ADR-010. Prefer this copy.

Decisions locked for the places product family. No secrets. No pixel UI or OpenAPI here.

## Product map

- what2eat (what2eat.food): thin web + BFF; restaurant UX; place/map via places-agent.
- where2play (where2play.place): thin web + BFF; trip UX; itinerary engine via places-agent.
- places-agent (places.agent-mate.ai): place tools, vendor adapters, agent LLM, HTTP + MCP, plan_itinerary, AND the operator management web on the same host.

Ownership: consumer screens and product OPENAI_CN stay on the apps (ADR-001). places-agent owns adapters, tool loop, itinerary planning, and operator UX (ADR-012). The admin app is not a fourth product.

```
Consumer browsers                         Operator browser
what2eat.food / where2play.place          places.agent-mate.ai
(thin web + same-origin BFF)              (admin HTML + same-origin BFF)
         |                                         |
         | caller API key                          | admin session
         +--------------------+--------------------+
                              v
              places-agent (one process)
              admin UI + HTTP API + MCP
                    |
         AMAP / Google Maps / Tripadvisor
```

## Runtime trust (four runtimes)

consumer browser | operator browser | app BFF | places-agent. Secrets never ship to either browser.

| Capability | Consumer browser | Operator browser | App BFF | places-agent |
| --- | --- | --- | --- | --- |
| OPENAI_CN OPENAI_* | No | No | Yes (product UX) | Yes (agent/tools) |
| AMAP / Google / Tripadvisor keys | No | No (not in admin UI) | No | Yes (env) |
| Admin session cookie | No | Yes same-origin | No | Issues/validates |
| Caller API key plaintext | No | Once at create/regenerate | Sends server-to-server | Stores hash; validates HTTP/MCP |
| HTTP tools | Same-origin app routes only | No (session ≠ caller key) | Server-to-server + caller key | Serves |
| MCP | N/A | N/A | Optional | Serves + caller key |
| Resend | No | Triggers via admin API | No | Yes (env) |

Flows:

- Operator: browser → places.agent-mate.ai same origin → session → users / caller keys / instructions → Resend.
- Place/map: consumer browser → app same-origin API → places-agent (caller key) → AMAP | Google | Tripadvisor.
- Product LLM: consumer browser → app BFF → OPENAI_CN (app OPENAI_*).
- Agent LLM: caller → BFF or MCP host → places-agent (caller key) → OPENAI_CN (agent OPENAI_*).

## places-agent core

Tools: search_restaurants, search_places, get_place_details, navigate; geocode as needed; plan_itinerary. HTTP and MCP share one tool core. Both require a caller API key.

Adapters: AMAP, Google Maps (direct REST, Cloudflare Worker MCP fallback — ADR-017), Tripadvisor. Cards: provider / sources[] / deep links.

Weather: Open-Meteo only (ADR-014). Not a providers[] vendor. Localize WMO codes.

Operator web (same process): public home, login, landing, admin users, caller keys, instructions, admin i18n. Auth = admin session. Public register off. Seed default admin on first deploy; do not bake a password into the image.

Caller identity: machine id places-agent (ADR-013). MCP serverInfo.name and HTTP field agent MUST be this string. Tool names stay unprefixed.

Data: admin users and caller-key hashes in SQLite+Prisma on the agent volume (ADR-015). Map-vendor keys, OPENAI_CN, Resend, session secret stay in env.

Process entry: server.ts (ADR-016). Dispatches /mcp and /sse+/messages; Next App Router handles HTML and /v1. Production CMD is node server.ts. No MCP sidecar.

## LLM posture

OPENAI_CN OPENAI_* on the server of that deployable. Not destination-routed. Optional other OPENAI_CN models in inventories are experiments, not a geo router.

## Provider gateway (caller-driven)

Supersedes mainland destination ⇒ AMAP only and geo-capability-route.json (deleted; do not recreate).

Caller passes providers[] (enrich, merge). Agent validates, calls adapters, tags results. Missing credentials / unsupported capability → explicit error or skip, not silent vendor swap. Mainland what2eat may request ["AMAP"]; HK/overseas may request ["GOOGLE_MAPS"] + Tripadvisor enrich — caller policy.

## Result model and client navigation

Cards carry provider and/or sources[] (id, logo URL, deep links). Optional merge → one cluster, primary_provider. Nav choice is client-side (e.g. mainland prefers AMAP deeplink when present). Agent returns all available secret-free links.

## Tripadvisor

Match by name + location. Never pass Google place_id. Best-effort; must not wipe primary search.

## Itinerary ownership

plan_itinerary engine: places-agent. Trip screens / save/share: where2play.

## Deployment (野草云3)

Three stacks, three images, one process per image. Host 野草云3 / svr_hk_vps_3. Pipeline GHCR → Portainer → Nginx Proxy Manager → Cloudflare.

places-agent: one hostname → one container for /, /v1, /mcp, /sse, /messages. Do not copy kb-agent NPM Custom Locations (that pattern is for two containers on one hostname).

Avoid: fourth places-admin stack; admin-nginx + agent-API as two processes in one container; map-vendor keys in the image or admin UI.

## Workspace remotes

Parent places-workspace: umbrella specs only (github.com/ethanhuangcst/places.workspace.git). Children (gitignored from parent): 0.1.sdd.sample, 0.2.release-bot, 1.places-agent, 2.what2eat, 3.where2play.

## ADR index (001–018)

ADR-001 thin-app/agent split. ADR-002 same-origin BFF. ADR-003 dual transport. ADR-004 OPENAI_CN per deployable. ADR-005 caller-driven providers. ADR-006 provenance + client nav. ADR-007 Tripadvisor no ID passthrough. ADR-008 itinerary engine vs trip UX. ADR-009 deploy Option 1. ADR-010 umbrella workspace. ADR-011 independent HK/TW. ADR-012 admin UI on agent. ADR-013 id places-agent. ADR-014 Open-Meteo. ADR-015 SQLite+Prisma. ADR-016 custom HTTP server. ADR-017 GMaps Worker MCP fallback. ADR-018 MVP by capability; all admin UI in MVP-1.
