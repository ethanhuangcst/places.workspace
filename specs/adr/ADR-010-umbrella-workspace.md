# ADR-010: Umbrella workspace vs child remotes

## Status

Accepted

## Decision

Parent git (`places-workspace`) holds umbrella specs only. Children — release-bot, sdd.sample, places-agent, what2eat, where2play — are separate remotes, gitignored from the parent.

## Consequences

Specs stay reviewable in one place; app code versions independently.
