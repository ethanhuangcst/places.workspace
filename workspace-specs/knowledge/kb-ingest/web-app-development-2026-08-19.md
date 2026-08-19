# Web app development — MVP-2 and integration guide addendum (2026-08-19)

Prefer this over the 2026-08-18 web-app-development slices for HTTP TC-H, AppChrome login loop, and the integration guide table. Project: places-workspace. No secrets.

## AppChrome login loop

Client `AppChrome` must not redirect to `/login` on a missing session. The **RSC layout** (`app/admin/layout.tsx`) owns the auth gate. Login RSC redirects to admin **only** when a user row already exists (closed register). `/login/fresh` remains the session-clear route.

## MVP-2 agent slice (2026-08-18)

- E2E `with_server.py` must poll `GET /v1/health` for `{ "agent": "places-agent", "ok": true }`, not just an open TCP port.
- `e2e/run.py` picks an ephemeral port when the preferred port is bound.
- Chat CI: `QUANZIL_MODE=fixture` (or missing `OPENAI_API_KEY`) drives deterministic tool-call turns.
- Six public MCP/HTTP tools + `/v1/chat`. Tripadvisor and Open-Meteo are server-side helpers. Chat and enrich are HTTP-only (ADR-020).

## HTTP user tests + integration guide (2026-08-19)

- Automate **TC-H01–H15** in `tests/http-tc-h.test.ts`; default `make test` gate. ChatBox **TC-C** deferred when HTTP is green (ADR-019).
- **TC-H12**: HTTP vs in-memory MCP in Vitest — do not rely on ChatBox for Feature 11.
- **TC-H15**: mocked Google live adapter in default CI; `make test-live` / `verify-gmaps-fallback.sh` with `GOOGLE_DIRECT_FORCE_FAIL=1` when `GMAPS_MCP_*` is set.
- Fixture: HK Japanese cuisine POIs for TC-H02/H03; generic `"restaurant"` filter must not hide specialty venues (TC-H14).
- Integration guide: copyable `.cursor/mcp.json` uses remote `url` + `headers`, not stdio. Multiline JSON needs `white-space: pre`.
- Capabilities table: merge Capability + Tool into **Capabilities**. First six cells are tool-name literals. Last two: Place chat (i18n) and `Tripadvisor.enrich` (dot, not JSON `enrich.tripadvisor`).
- Channel: six tools `HTTP and MCP`; HTTP-only rows `HTTP only` (no parentheses). Place chat also shows `POST /v1/chat`.
- Hairlines: last table-row border + next `h2` top border stacks. Drop last-row rule; keep one section divider. TOC has no bottom rule.
- Nested git: workspace Grep/Glob often miss `1.places-agent/` (own `.git`).
- `.env.production.example` for Portainer — exclude `GOOGLE_DIRECT_FORCE_FAIL`, `DEV_ADMIN_PASSWORD`, `ALLOWED_DEV_ORIGINS`, `TEST_DATABASE_URL`, `QUANZIL_MODE`.
- Local server: `make dev` in a kept-open terminal; `make up` from short-lived shells often dies after health OK.
- MCP clients: Cursor → `/mcp`; ChatBox → `/sse` + Bearer. Name `places-agent`.
