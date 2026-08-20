# what2eat MVP-2 — lessons

Captured after MVP-2 sign-off (Decide, place details, Saved) on 2026-08-19.

## What shipped

- Decide page mock-aligned: `.decide-form` grid, meal-context datalist, results head, reshuffle, pagination, pick cards, empty/partial states.
- `PickCard` and `PlaceDetailsDialog` components with live `data-testid`s per [`2eat-design.md`](../../2.what2eat/2eat-specs/2eat-design.md) §3。
- Saved page: `.pick-grid`, Details dialog (unsave variant), empty state.
- i18n: `eat.card.*`, `eat.match.fit.*`, `eat.details.*` (removed ad-hoc `eat.pick.*` / `eat.place.*` usage).
- BFF client: empty agent HTTP bodies no longer crash JSON parse.

## Live probe

- `make test-e2e-mvp2-live` green with places-agent on `:3010`, valid `PLACES_AGENT_CALLER_KEY`, and `PLACES_AGENT_TIMEOUT_MS=90000` (geocode + dual-provider search can exceed 25s).
- Probe pin: Clerkenwell, London; cards from GOOGLE_MAPS; no `fixture_` native ids.

## Operator checklist (local)

1. Postgres `:5435` up (`make up` in what2eat).
2. places-agent: `npx prisma migrate deploy` (CallerApiKey table) + issue caller key (`npx tsx scripts/issue-caller-key.ts`) if auth returns 401.
3. what2eat `.env.local`: `PLACES_AGENT_BASE_URL`, `PLACES_AGENT_CALLER_KEY` matching an ACTIVE agent key; consider `PLACES_AGENT_TIMEOUT_MS=90000` for live search.
4. Start agent (`PORT=3010 npm run dev`), then `make test-e2e-mvp2-live`.

## Deferred (MVP-3)

- Agent chat FAB/panel on Decide (`decide-07`).
- Place chat block inside details dialog (`place-03`).
- History page (`history-01`).

## ADR

No new ADR — MVP-2 follows ADR-020 (HTTP-only) and ADR-021 (live vendor, no fixture ids).
