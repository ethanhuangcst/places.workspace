# KB ingest pack — where2play MVP-1 close (2026-08-21)

Project: **places-workspace** / product **where2play**. No secrets. Product AC stays in `3.where2play/2play-specs/`.

## Gaps proposed (2026-08-21)

| Item | Source | Why ingest |
| --- | --- | --- |
| ADR-033 | `workspace-specs/adr/ADR-033-where2play-postgres-prisma.md` | where2play dedicated Postgres; thin-app LLM boundary |
| Next env + CSS | `3.where2play/2play-specs/knowledge/next-env-css-mvp1.md` | Empty SESSION_SECRET override; Tailwind `@import` 500 |
| MVP-1 quality gate | `3.where2play/2play-specs/knowledge/mvp1-quality-gate.md` | `make quality` coverage + with_server reuse |
| MVP-1 close | `3.where2play/2play-specs/knowledge/where2play-mvp1-close.md` | Slice close + usability confirm |

## Do not ingest

- `.env.local` / `.env.production` values
- Full `ui-mockup/` HTML
- Story AC tables (rot)

## Confirm

User requested propose + confirm in the same turn (`kb_import_documents` → `kb_confirm_import_batch` with `confirm_all_viable=true`).

**Indexed** batch `42c5bc82-13f0-4c47-a6e6-65df62bc8e74`:

| kb title | knowledge_id |
| --- | --- |
| ADR-033: PostgreSQL + Prisma for where2play | `85d7d564-6efc-49f7-b183-19d2217cab48` |
| where2play MVP-1 close | `0b8b9815-d298-4d4a-b457-337fb327f5a2` |
| Next.js env + CSS gotchas (where2play MVP-1) | `6aeb8640-a8ff-4ac6-835d-0f29ee7ff4e2` |
| MVP-1 quality gate | `e64ccf83-6bf0-4184-8a53-52bf2e4c185f` |
