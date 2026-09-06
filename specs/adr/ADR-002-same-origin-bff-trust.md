# ADR-002: Browser → same-origin BFF → agent/vendors

## Status

Accepted

## Decision

Secrets never ship to the browser. Map / Tripadvisor keys live only on places-agent. Apps may hold OPENAI_CN (`OPENAI_*`) on their BFF for product UX. Browser talks only to same-origin app APIs (or opens secret-free deep links / logos).

## Consequences

Clear trust boundary; callers authenticate to places-agent server-to-server (or via MCP host).
