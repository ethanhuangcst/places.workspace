# ADR-004: OPENAI_CN fixed per deployable

## Status

Accepted — wording updated 2026-08-23 ([ADR-041](./ADR-041-openai-cn-replaces-quanzil.md); formerly “Quanzil”)

## Decision

Each deployable that needs LLM configures its own **OPENAI_CN** OpenAI-compatible gateway via `OPENAI_*` on the server. LLM is not routed by search destination. Gateway host is **not** fixed in specs — operators set `OPENAI_BASE_URL` per environment.

## Consequences

Simpler ops than geo-coupled model switching; product vs agent `OPENAI_*` keys stay separate.
