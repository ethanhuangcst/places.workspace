# ADR-033: PostgreSQL + Prisma for where2play

## Status

Accepted

## Context

where2play persists users, interest profiles, plan session cache, saved itineraries, and **chat snapshots committed only on Save** (draft chat stays in browser localStorage — see [`2play-prod-specs.md`](../../3.where2play/2play-specs/2play-prod-specs.md) and [`2play-design.md`](../../3.where2play/2play-specs/2play-design.md) §2).

Sister thin app what2eat already uses PostgreSQL + Prisma ([ADR-023](./ADR-023-what2eat-postgres-prisma.md)). places-agent uses a dedicated Postgres db `places_agent` ([ADR-025](./ADR-025-places-agent-postgres-prisma.md)). Early where2play deployment drafts left persistence **TBD** and warned against SQLite-on-volume.

where2play is a consumer app with relational account/trip data and the same Portainer “web-only, DB off-node” ops pattern as what2eat.

## Decision

1. **where2play uses PostgreSQL + Prisma** — not SQLite.
2. **Local dev:** PostgreSQL. Dedicated database **`where2play`** (and **`where2play_test`** for tests). Default: reuse the family local instance on **`localhost:5435`** (same host bind as what2eat `5435→5432`). Never store where2play rows in `what2eat` or `places_agent`.
3. **Production:** external Postgres on Aliyun **`101.132.156.250:5432`**, database **`where2play`** (dedicated). Do **not** reuse `what2eat`, `places_agent`, `kb_agent`, `mypoke_trade_prod`, `media_marketing`, or `hca`.
4. **No SQLite / data volume** on the Portainer `where2play` stack for the primary store. One web container; Postgres off-node.
5. **Migrate on boot** in the prod image entrypoint (`prisma migrate deploy`) after `DATABASE_URL` is reachable.
6. **Tests** use `TEST_DATABASE_URL` (db `where2play_test` or isolated schema), never production credentials.
7. **Chat persistence:** only messages included in `POST /api/saved` are written to App DB; there is no per-turn chat table write path.

## Rationale

- Same engine and ops as what2eat (ADR-023); one Prisma `provider = "postgresql"` for local and prod.
- Separates consumer trip data from places-agent admin store and from what2eat dining data.
- Matches release-bot semi-auto release: main DB off-node, Portainer stack stays one web container.

**Rejected:** SQLite on a `where2play_data` volume; sharing `what2eat` / `places_agent` database names; embedding Postgres in the where2play image.

## Consequences

- `3.where2play/prisma/schema.prisma` uses `provider = "postgresql"` (when the app lands).
- Local example: `DATABASE_URL=postgresql://where2play:where2play@localhost:5435/where2play`.
- Test example: `TEST_DATABASE_URL=postgresql://where2play:where2play@localhost:5435/where2play_test`.
- Prod: `DATABASE_URL=postgresql://…@101.132.156.250:5432/where2play` (Portainer-only secret). Operator must **create the empty database** on Aliyun before first migrate.
- Child plan [`2play-deployment-plan.md`](../../3.where2play/2play-specs/2play-deployment-plan.md) §5–§6: `DATABASE_URL` **required**; §6 Engine = PostgreSQL (this ADR).
- Family [`6.deployment-plan.md`](../6.deployment-plan.md): where2play persistence no longer TBD.
- where2play **does not** use product `OPENAI_*` ([`2play-design.md`](../../3.where2play/2play-specs/2play-design.md) §2.1); LLM stays on places-agent.

## Date

2026-08-21
