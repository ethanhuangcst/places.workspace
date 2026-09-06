# ADR-028: Record decision history server-side on save

## Status

Accepted

## Context

MVP-3 **History** (`history-01`) lists `DecisionHistory` rows (place snapshot, area, meal context, outcome `went`). Initial implementation recorded history **client-side** via `recordWent()` → `POST /api/history`, with errors swallowed (`best-effort`).

Live E2E (`test_mvp3_live.py`) clicks **Save** then immediately **Close** on the place dialog, then expects `[data-testid="history-row"]`. That race often left **no server row** even when save succeeded.

A second failure mode on **repeat runs**: if the place was already in `SavedPlace`, the Decide dialog opened with `saved: true` and the same `place-save` button **unsaved** on click — removing the favorite and still not guaranteeing history.

## Decision

1. **`POST /api/saved`** creates a matching **`DecisionHistory`** row in the same transaction path (place snapshot, optional `area`, optional `mealContext`, outcome `went`).
2. On Decide, **`place-save` is save-only** — it never unsaves from that control; already-saved places show **Saved** and a second click calls `recordWent()` to backfill history if missing.
3. **`recordWent()`** remains for **Open map** and other went signals; save path no longer depends on a follow-up client history POST.
4. **`AgentChatPanel`** renders the chat composer only when the panel is **open**, so closed list-chat DOM does not expose hidden `[data-testid="chat-agent-msg"]` nodes that block place-chat E2E waits.

Unsave stays on Saved page / saved-variant dialog only.

## Rationale

- History is account data (PostgreSQL, ADR-023); tying it to the successful save mutation is reliable under fast UI/E2E timing.
- Save-only Decide button avoids accidental unsave on repeat live probes and matches E2E intent (one save click → history row).
- Composer mount guard is a testability fix aligned with UX (no hidden inputs when panel closed).

**Rejected:** changing E2E to wait for async client history; relying solely on `keepalive` fetch after dialog unmount.

## Consequences

- `tests/api/saved.test.ts` asserts history after save.
- Duplicate history rows possible if user saves again when already saved (acceptable for MVP-3; dedupe deferred).
- Product specs unchanged: History still **went-only**; chat remains browser-local (ADR-020).

## Date

2026-08-20
