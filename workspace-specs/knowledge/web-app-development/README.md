# Web app development

Reusable lessons from building operator-facing admin web apps on Next.js (first evidence: **places-agent MVP-1**, 2026-08-18). Not product AC — engineering process, failure modes, and test gates.

Product requirements stay in app `agent-specs/` / `req-specs`. Binding architecture stays in `../adr/`.

## Index

| Doc | Topic | Updated |
| --- | --- | --- |
| [lessons-from-places-agent-mvp1.md](./lessons-from-places-agent-mvp1.md) | **Consolidated** auth forms, Next 16 constraints, E2E, cross-machine dev, mail, DoD gates | 2026-08-18 |

## Superseded / merged sources

These ops notes are folded into the consolidated doc; keep them as short pointers only:

| Old path | Status |
| --- | --- |
| [../ops/places-agent-admin-invite-dev.md](../ops/places-agent-admin-invite-dev.md) | Merged → § Cross-machine dev, § Auth forms |
| [../ops/places-agent-next-runtime.md](../ops/places-agent-next-runtime.md) | Merged → § Next.js App Router |

## KB ingest

Propose-only copies for **kb-agent** live under [`../kb-ingest/`](../kb-ingest/README.md). Writes are **propose → human confirm**. See the ingest manifest in [`../kb-ingest/web-app-development-2026-08-18.md`](../kb-ingest/web-app-development-2026-08-18.md).
