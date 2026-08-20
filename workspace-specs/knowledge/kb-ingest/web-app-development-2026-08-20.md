# KB ingest pack — what2eat MVP-3 close (2026-08-20)

Project: **places-workspace**. No secrets. Product AC stays in `2.what2eat/2eat-specs/`.

## Gaps proposed (2026-08-20)

| Item | Source | Why ingest |
| --- | --- | --- |
| ADR-021 | `adr/ADR-021-live-vendor-no-fixture.md` | KB has ADR-020; 021 live honesty gate missing |
| ADR-022 | `adr/ADR-022-timed-itinerary.md` | Timed itinerary contract not in kb |
| ADR-023 | `adr/ADR-023-what2eat-postgres-prisma.md` | what2eat Postgres decision |
| ADR-024 | `adr/ADR-024-quality-gates-typescript-7.md` | TS7 lint/coverage/E2E sidecar |
| ADR-025 | `adr/ADR-025-places-agent-postgres-prisma.md` | Supersedes ADR-015 for agent |
| ADR-026 | `adr/ADR-026-region-based-provider-auto-selection.md` | Provider auto-selection; Taiwan excludes AMAP |
| ADR-027 | `adr/ADR-027-decide-searchcache-hydrate.md` | **New** — reload hydrate API |
| ADR-028 | `adr/ADR-028-decision-history-on-save.md` | **New** — history on save |
| what2eat MVP-3 lessons | `knowledge/web-app-development/what2eat-mvp3-lessons.md` | E2E failure modes + operator checklist |
| Architecture refresh 2026-08-20 | `kb-ingest/architecture-2026-08-20.md` | ADR index through 028 |

## Do not ingest

- `2.what2eat/2eat-specs/` product stories (rot quickly)
- `.env.local` values
- Full handbook dump (prefer ADR deltas until handbook refresh decided)

## Prefer over older kb copies

- Architecture refresh **2026-08-20** over 2026-08-19 (adds ADR-021–028 one-liners)
- ADR-026 over ADR-005-only routing in older handbook sections
