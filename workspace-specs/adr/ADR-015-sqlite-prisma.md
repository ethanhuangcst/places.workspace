# ADR-015: SQLite + Prisma on the places-agent volume

## Status

**Superseded** by [ADR-025](./ADR-025-places-agent-postgres-prisma.md) (2026-08-19). Do not use SQLite for new places-agent work.

## Context

places-agent must persist admin users and hashed caller API keys (ADR-012). Alternatives: local JSON (forbidden by `3.tech-specs.md` for this store), Postgres as a fourth process, or SQLite on the existing stack volume. One writer process, tens of rows, small team, 野草云3 Option 1 (three stacks, not four).

## Decision

Use **SQLite (WAL) + Prisma** on a volume attached to the places-agent container. `DATABASE_URL=file:/data/places-agent.db` (or equivalent). Migrate + seed on boot. Seed username `admin`, email `me@ethanhuang.com`; do not bake a password into the image.

## Rationale

JSON is not a durable source of truth for credentials. Postgres is the right move for multiple writers or HA — that is not MVP. SQLite matches one Node process and one volume backup.

Rejected: JSON files; a separate Postgres stack for this deployable.

## Consequences

- Prisma schema lives in `1.places-agent`. Volume must be in the Portainer stack.
- Moving to Postgres later is a **new ADR** when we run more than one writer.
- Tests use an isolated SQLite file, never production `/data`.

## Date
2026-08-17
