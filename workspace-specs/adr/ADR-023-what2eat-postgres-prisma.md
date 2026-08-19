# ADR-023: PostgreSQL + Prisma for what2eat

## Status

Accepted

## Context

what2eat persists users, taste profiles, saved places, and decision history (not chat — browser localStorage only). Early drafts copied places-agent’s SQLite-on-volume pattern ([ADR-015](./ADR-015-sqlite-prisma.md)). ADR-015 applies **only** to places-agent (one writer, tens of admin rows).

what2eat is a consumer app with relational profile/saved/history data, likely to grow beyond a single-file SQLite backup, and aligns with the family’s existing **external Postgres** ops pattern on Aliyun (`101.132.156.250`) used by kb-agent and mypoke-trade.

## Decision

1. **what2eat uses PostgreSQL + Prisma** — not SQLite.
2. **Local dev:** dedicated Postgres via `docker-compose.dev.yml` (or operator-managed Postgres). Host bind **`5435→5432`** by default to avoid clashing with other local Postgres on `5432` / `5434`.
3. **Production:** external Postgres on Aliyun **`101.132.156.250:5432`**, database **`what2eat`** (dedicated; do not reuse `kb_agent`, `mypoke_trade_prod`, `media_marketing`, or `hca`).
4. **Migrate on boot** in the prod image entrypoint (`prisma migrate deploy`) after `DATABASE_URL` is reachable.
5. **Tests** use an isolated database (`TEST_DATABASE_URL` or per-run schema/db), never production credentials.

## Rationale

- Separates consumer data from places-agent’s SQLite admin store.
- Matches release-bot semi-auto release: main DB off-node, Portainer stack stays one web container.
- Prisma stays the ORM; only the datasource provider changes.

**Rejected for what2eat:** SQLite file on `what2eat_data` volume (places-agent pattern); sharing kb/mypoke DB names.

## Consequences

- `prisma/schema.prisma` uses `provider = "postgresql"`.
- `2.what2eat/.env.example` documents `DATABASE_URL` / `TEST_DATABASE_URL` as `postgresql://…`.
- Family [`6.deployment-plan.md`](../6.deployment-plan.md) drops `what2eat_data` SQLite volume; prod `DATABASE_URL` is Portainer-only secret.
- places-agent Postgres is a separate decision: [ADR-025](./ADR-025-places-agent-postgres-prisma.md) (dedicated db `places_agent`, local port `5436`). Do not share `what2eat`.

## Date

2026-08-19
