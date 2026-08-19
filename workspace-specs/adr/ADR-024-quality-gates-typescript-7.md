# ADR-024: Quality gates on TypeScript 7 (ESLint syntax, coverage, isolated E2E)

## Status
Accepted

## Context

places-agent runs **TypeScript 7.0** and Next 16. `eslint-config-next/typescript` loads `typescript-eslint`, which **refuses TS 7.0** at import time. `make lint` had been aliased to `tsc --noEmit`, so lint was not a real gate. Coverage was unmeasured. Admin Playwright used `with_server.py` with stdout/stderr piped and the same Next `distDir` as a running `make up`, so a second `next dev` died on the existing lock (PID on port 3010) even when E2E asked for port 3000.

## Decision

1. **Keep TypeScript 7.** Do not pin 5.x to please `typescript-eslint`.
2. **`make typecheck`** = `npx tsc --noEmit`. **`make lint`** = `npx eslint .` with `@babel/eslint-parser` + `@babel/preset-typescript` / React, plus `@next/eslint-plugin-next` core-web-vitals. No type-aware ESLint until `typescript-eslint` supports TS 7.
3. **`make test-coverage`**: Vitest v8; include `src/**/*.{ts,tsx}`; overall statements/lines/functions/branches **≥ 80%**. Exclude list is explicit in [`1.places-agent/agent-specs/4.test-strategy.md`](../../1.places-agent/agent-specs/4.test-strategy.md) (admin UI via Playwright, Worker MCP client, env/`live` wiring, session/mail, NL loop). Core floors: `place-filters.ts` 100%; other listed core files **≥ 90% lines**.
4. **`make test`** stays fixture Vitest only (no browser, no coverage).
5. **`make quality`** = typecheck + lint + test-coverage + test-e2e.
6. **Admin E2E sidecar:** do not pipe Next logs into unread `PIPE` (deadlock). Set `NEXT_DIST_DIR=.next-e2e` so the lock file is not `.next/dev/lock` of the operator’s 3010 process. Set `QUANZIL_MODE=fixture` so `/v1/chat` smoke does not require a live LLM token. Do not kill the operator’s `make up`.

## Rationale

- Typecheck and lint are different jobs; faking lint with `tsc` hid Next HTML-link rules.
- Type-aware lint on a patched `typescript.versionMajorMinor` crashed on the TS 7 API.
- Coverage without an exclude list would fail 80% on admin UI and MCP client; silently excluding `app/` to inflate numbers is forbidden — the list is in the test strategy.
- Next 16 treats one `distDir` as a single running app; a second PORT is not enough.

Rejected: pin TypeScript 5; disable whole ESLint rules to go green; lower the 80% overall floor; reuse the live 3010 process for Playwright.

## Consequences

- Babel lint is **syntax + Next vitals**, not typed rules. `tsc` remains the type gate.
- E2E first compile against `.next-e2e` is slower; gitignore `.next-e2e/`.
- Critical-path **100% on every metric** for itinerary/tools is not yet the Vitest floor (lines ≥90% there). Raising those files to 100% is a follow-up, not a reason to drop the overall 80% gate.

## Date
2026-08-19
