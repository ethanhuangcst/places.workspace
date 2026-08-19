# ADR-019: HTTP-first automated user test cases (ChatBox manual deferred)

## Status

Accepted

## Context

places-agent exposes the same tools on **HTTP** (`/v1/*`) and **MCP** (`/mcp`, `/sse`) per ADR-003. Feature 11 requires semantic parity across channels. [`8.user-test-cases.md`](../../1.places-agent/agent-specs/8.user-test-cases.md) defined manual **TC-H** (HTTP) and **TC-C** (ChatBox MCP) cases for release sign-off.

Running ChatBox manual QA in CI is brittle (external client, model variance, operator steps). HTTP cases map 1:1 to contract tests. MCP parity still must be proven without relying on ChatBox alone.

## Decision

1. Automate **TC-H01–H15** exactly in [`tests/http-tc-h.test.ts`](../../1.places-agent/tests/http-tc-h.test.ts); include in default **`make test`** / PR gate.
2. Prove HTTP↔MCP tool parity in **TC-H12** via in-memory MCP transport in the same Vitest suite — not via ChatBox.
3. **Defer ChatBox manual TC-C** sign-off when the HTTP equivalent is green in CI. ChatBox remains for ad-hoc client regression and operator demos, not the default merge gate.
4. **TC-H15** (Google direct → Worker MCP fallback): mocked adapter in default CI; opt-in **`make test-live`** / `verify-gmaps-fallback.sh` when `GMAPS_MCP_*` is configured.

## Rationale

- HTTP harness is deterministic, fast, and runs in every push.
- TC-H12 satisfies Feature 11 without an MCP host UI.
- ChatBox adds value for SSE client quirks (`/sse` vs `/mcp`) but duplicates tool semantics already covered by HTTP + in-memory MCP.
- Live Worker fallback stays opt-in to avoid paid/egress flakiness in default CI (ADR-017).

**Rejected:** ChatBox-only sign-off; dropping MCP parity checks; running live Google/Worker in default CI.

## Consequences

- [`4.test-strategy.md`](../../1.places-agent/agent-specs/4.test-strategy.md) and [`8.user-test-cases.md`](../../1.places-agent/agent-specs/8.user-test-cases.md) must state HTTP automation + ChatBox pause.
- New tools need HTTP contract tests; add MCP parity test when Feature 11 applies.
- Release sign-off may still require a periodic ChatBox smoke on `/sse` before major releases — documented as manual, not CI-blocking.

## Date

2026-08-19
