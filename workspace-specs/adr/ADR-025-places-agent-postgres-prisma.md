# ADR-025: PostgreSQL + Prisma for places-agent

## Status

Accepted

## Context

[ADR-015](./ADR-015-sqlite-prisma.md) stored admin users and hashed caller API keys in SQLite on a Docker volume. what2eat already uses PostgreSQL ([ADR-023](./ADR-023-what2eat-postgres-prisma.md)). Prisma cannot target SQLite in production and PostgreSQL locally from one schema. The operator chose one engine for both environments: dedicated Postgres locally and a dedicated Aliyun database in production.

## Decision

1. **places-agent uses PostgreSQL + Prisma** — not SQLite. ADR-015 is **superseded**.
2. **Local dev:** PostgreSQL. Dedicated databases `places_agent` and `places_agent_test`. Default: reuse the what2eat local instance on **`localhost:5435`**. Optional isolated compose: `1.places-agent/docker-compose.dev.yml` on **`5436→5432`**. Never store agent rows in the `what2eat` database.
3. **Production:** external Postgres on Aliyun **`101.132.156.250:5432`**, database **`places_agent`** (dedicated). Do not reuse `what2eat`, `kb_agent`, `mypoke_trade_prod`, `media_marketing`, or `hca`.
4. **No SQLite volume** (`places_agent_data` / `/data/places-agent.db`) on the Portainer stack. The web container stays one process; Postgres is off-node (same ops pattern as what2eat).
5. **Migrate + seed on boot.** Tests use `TEST_DATABASE_URL` (`places_agent_test`), never the production Aliyun URL.

## Rationale

One Prisma `provider = "postgresql"` for local and prod. Aligns family ops (external Postgres, Portainer web-only). Admin data is small; the change is for engine consistency and backup/ops, not multi-writer HA.

**Rejected:** dual SQLite+Postgres schemas; sharing the `what2eat` database; putting Postgres as a second process in the places-agent image.

## Consequences

- `prisma/schema.prisma` uses `provider = "postgresql"`.
- Local `DATABASE_URL=postgresql://places_agent:places_agent@localhost:5435/places_agent` (or `:5436` with the places-agent compose).
- Prod `DATABASE_URL=postgresql://…@101.132.156.250:5432/places_agent` (Portainer secret). Operator must create the empty database on Aliyun before first migrate.
- Family [`6.deployment-plan.md`](../6.deployment-plan.md) drops the SQLite volume for this stack.
- Existing local SQLite files under `1.places-agent/.data/*.db` are leftover; do not treat them as source of truth. Re-seed admin after first Postgres migrate.

## Date

2026-08-19
