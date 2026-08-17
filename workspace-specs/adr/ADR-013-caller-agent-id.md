# ADR-013: Caller-visible agent id is `places-agent`

## Status
Accepted

## Context

Callers (what2eat / where2play BFFs, ChatBox and other MCP hosts, operator instructions) need one stable string to know which service they reached. Candidates were the hostname `places.agent-mate.ai`, a display/marketing title, `place-agent`, `what2eat-agent`, or the machine id `places-agent` already used in specs and the repo folder.

## Decision

1. The **caller-visible agent id** is the exact string **`places-agent`**. It is a protocol identifier: not localized, not translated, not inferred from the hostname.
2. **MCP:** `initialize` result `serverInfo.name` MUST be `places-agent`.
3. **HTTP:** tool responses and the health/ready document MUST include JSON field `agent` with value `places-agent`.
4. **Instructions** (Feature 18) MUST tell operators to register / select this MCP server as `places-agent`. ChatBox (or other) **display** titles MAY differ; the machine id must not.
5. Tool names stay capability names (`search_restaurants`, …). Do **not** prefix tools with `places-agent_` (that would fork HTTP vs MCP names).
6. Hostname `places.agent-mate.ai` remains the **host**, not the agent id. Env `APP_NAME`, Portainer stack name, and MCP server id stay `places-agent`.

## Rationale

- One id across HTTP, MCP, and docs so BFFs and agent hosts can assert they hit this service, not what2eat or a renamed ChatBox label.
- Hostname can change; the machine id should not.
- Tool names are the capability contract (ADR-003); identity belongs on the server, not on every tool.

Rejected: using the hostname as the id; `place-agent`; per-product ids (`what2eat-agent`).

## Consequences

- Tests assert the literal `places-agent` on MCP initialize and HTTP `agent`.
- i18n catalogs must not replace this string.
- Follow-up if a public rename is ever needed: new ADR; do not silently change the id.

## Date
2026-08-17
