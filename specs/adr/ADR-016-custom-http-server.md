# ADR-016: Custom Node HTTP server as places-agent entry

## Status
Accepted

## Context

One process must serve operator HTML, admin APIs, HTTP tools, and MCP (ADR-012). The MCP TypeScript SDK and ChatBox legacy SSE expect Node `IncomingMessage` / long-lived streams. Next.js App Router `Request` handlers are a poor fit for `/sse`. A second MCP container would violate one image / one process.

## Decision

1. Process entry is **`server.ts`**: dispatch `/mcp` (Streamable HTTP) and `/sse` + `/messages` (SSE) in-process; pass all other URLs to the Next.js request handler.
2. `/v1` tools, `/api/admin`, and operator pages stay **Next App Router**.
3. Docker `CMD` is `node server.ts` (standalone output). `next start` is not the production entry.
4. MCP sidecar is **forbidden**.

## Rationale

REST, Zod, and Vitest stay in Route Handlers. MCP transports stay where the SDK works. Operators still get one Portainer stack and one hostname.

Rejected: Next-only MCP; nginx + Node as two processes in one image; a `places-mcp` sidecar.

## Consequences

- Maintain a small `server.ts`. Local `make dev` must run this entry, not only `next dev`, once MCP is wired (pages-only stories may use `next dev` until `/mcp` exists).
- Next middleware must not attach to `/mcp`, `/sse`, `/messages`, or `/v1`.

## Date
2026-08-17
