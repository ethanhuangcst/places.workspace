# ADR-003: Dual transport over one tool core

## Status

Accepted

## Decision

Expose the same places-agent tools via HTTP API and MCP without forking business logic.

## Consequences

First-party BFFs and third-party agent hosts (e.g. chatboxai) share semantics; adapters stay single-sourced.
