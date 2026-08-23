# where2play trip assistant BFF OPENAI_CN (chat-01)

**Date:** 2026-08-22  
**Related:** [ADR-036](../../adr/ADR-036-where2play-assistant-quanzil.md), `POST /api/chat`

## Lessons

1. **Patch contract:** Prefer `itineraryPatch` (merge by `dayIndex`); else full `itinerary` replace; else transcript-only. Encode this in AC and `applyAssistantItineraryResult` so UI/BFF stay single-semantic.
2. **Do not NDJSON-stream raw model JSON to the bubble.** Collect OPENAI_CN completion, `parseAssistantModelText`, then emit `token` events for `reply` characters + `done` with patched itinerary — avoids showing `{ "reply":...` in the UI.
3. **protect-eng:** `.env.example` may list empty `OPENAI_*` after product ADR; never write the operator’s `.env.local` without confirmation. Missing key → `errors.openai_not_configured` (503).
4. **No agent `/v1/chat` on this path.** Tests assert fetch never hits `/v1/chat`; optional `search_*` remains future work when the model needs a venue swap.
