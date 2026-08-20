# Architecture — places workspace (refresh 2026-08-20)

Delta over 2026-08-19 refresh. Full body: `workspace-specs/2.architecture.md`. Project: places-workspace.

## Four runtimes (unchanged)

Consumer browser (what2eat / where2play) → same-origin BFF → places-agent HTTP/MCP → vendor APIs. Operator browser on places-agent host. Chat transcripts stay browser-local on what2eat (ADR-020); account data in Postgres (ADR-023).

## ADR index (001–028) — new since 2026-08-19

| ID | Title | One-liner |
| --- | --- | --- |
| ADR-021 | Live vendor no fixture | `PLACES_VENDOR_MODE=live` fail-closed; no `fixture_` ids; DoD honesty gate |
| ADR-022 | Timed itinerary | `plan_itinerary` `detail: timed`; meals, legs, weather buffers |
| ADR-023 | what2eat Postgres | Consumer app PostgreSQL + Prisma; local `:5435`; Aliyun db `what2eat` |
| ADR-024 | Quality gates TS 7 | Babel ESLint + `tsc`; Vitest coverage ≥80%; E2E `.next-e2e` sidecar |
| ADR-025 | places-agent Postgres | Supersedes ADR-015 SQLite; db `places_agent`; local `:5435` or `:5436` |
| ADR-026 | Region provider auto-select | Agent picks providers by region; Taiwan excludes AMAP; caller override kept |
| ADR-027 | Decide SearchCache hydrate | `GET /api/decide/current` restores reload; no re-search |
| ADR-028 | Decision history on save | `POST /api/saved` writes `DecisionHistory`; Decide save-only |

ADR-001–020 unchanged from 2026-08-19 architecture refresh in kb.

## what2eat MVP-3 additions (2026-08-20)

- `SearchCache` read on reload reconnects list chat `searchId` (localStorage keys unchanged).
- History (`DecisionHistory`) server-side on save; not chat.
- Live probe: `make test-e2e-mvp3-live` (Clerkenwell, London pin).
