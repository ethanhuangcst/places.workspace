# ADR-004: Quanzil fixed per deployable

## Status

Accepted

## Decision

Each deployable that needs LLM configures its own OpenAI Quanzil (`OPENAI_*`) on the server. LLM is not routed by search destination.

## Consequences

Simpler ops than geo-coupled model switching; product vs agent Quanzil keys stay separate.
