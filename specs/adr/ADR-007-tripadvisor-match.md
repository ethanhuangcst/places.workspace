# ADR-007: Tripadvisor enrichment without ID passthrough

## Status

Accepted

## Decision

Enrich by name + location match. Never forward Google `place_id` (or Google-native ids) to Tripadvisor as a place identifier.

## Consequences

Avoids invalid API use and id leakage across vendors; enrichment is best-effort.
