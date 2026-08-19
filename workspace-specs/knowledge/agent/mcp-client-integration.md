---
title: MCP client integration — Cursor and ChatBox
type: ops-lesson
status: active
as_of: 2026-08-19
tags:
  - places-agent
  - mcp
  - cursor
  - chatbox
related_spec: 1.places-agent/agent-specs/6.agent-deployment.md
related:
  - ../../adr/ADR-003-dual-transport.md
  - ../../adr/ADR-019-http-first-user-test-automation.md
  - ../ops/places-agent-local-daemon.md
---

# MCP client integration — Cursor and ChatBox

## Summary

places-agent serves MCP on the **same origin** as HTTP tools. Client configuration differs by host: Cursor uses **Streamable HTTP** at `/mcp`; ChatBox uses **Remote HTTP/SSE** at `/sse`. Remote configs use `url` + `headers` — not local stdio `command` / `args`.

## Evidence

- Routing in `1.places-agent/server.ts`: `/mcp` → Streamable HTTP; `/sse` → GET opens SSE, POST may initialize MCP (ChatBox pattern).
- Operator integration guide: `app/instructions` (`GuideCopyBlock`, i18n keys under `admin.guide.*`).
- Cursor remote MCP docs: `mcpServers.<name>.url` + optional `headers`; env interpolation `${env:VAR}`.
- Manual tests: ChatBox must not point at `/mcp`; header `Authorization: Bearer <caller_api_key>`.
- Remote local testing: `localtunnel --port 3010` exposes HTTPS for ChatBox on another device; first visit may show loca.lt interstitial.

## Lesson / guidance

### Cursor (remote Streamable HTTP)

File: project **`.cursor/mcp.json`** (or Settings → MCP → Edit config).

```json
{
  "mcpServers": {
    "places-agent": {
      "url": "https://places.agent-mate.ai/mcp",
      "headers": {
        "Authorization": "Bearer ${env:PLACES_AGENT_CALLER_KEY}"
      }
    }
  }
}
```

- Set `PLACES_AGENT_CALLER_KEY` in the shell environment, **or** use a literal `Bearer pa_…`.
- Reload Cursor after saving.
- Local dev: `http://localhost:3010/mcp` (match `PORT` in `.env.local`).

### ChatBox (Remote HTTP/SSE)

| Field | Value |
| --- | --- |
| Type | Remote (HTTP/SSE) |
| URL (prod) | `https://places.agent-mate.ai/sse` |
| URL (local) | `http://localhost:3010/sse` |
| Header | `Authorization: Bearer <caller_api_key>` |

Enable MCP tools on the chat. The chat model is ChatBox’s, not places-agent’s Quanzil loop.

Name the connection **`places-agent`**. **Edit MCP Server listing tools ≠ this chat has them.** ChatBox injects tools per conversation. An old thread started before MCP was enabled (or with only kb MCP on) will keep saying `search_restaurants` is missing. Fix: **new chat** → turn on MCP for that chat → enable the `places-agent` server → confirm the composer/tool panel lists `search_restaurants`. Do not ask the model to “self-check” the connection; look at that session’s tool list.

In one chat, do not also enable a vendor Maps MCP (the Cloudflare Worker at `GMAPS_MCP_*`) or a generic web-search / kb-search MCP — ChatBox’s model chooses among **all** enabled tools. `GMAPS_MCP_*` is places-agent’s **server-side** Google fallback (ADR-017), not a ChatBox client.

Header field may wrap visually; the value must still be `Authorization=Bearer pa_…` (one logical line). After changing MCP tool descriptions or the server name, Save, Test, then **new chat**.

### UI / copy pitfalls

- JSON snippets in the integration guide must render with **`white-space: pre`** (`.code-block--multiline`) or operators see one truncated line and assume wrong format.
- Protocol id **`places-agent`** is literal in MCP `serverInfo.name` and HTTP `agent` — not i18n.

### Testing without ChatBox

Default CI uses HTTP TC-H01–H15 + in-memory MCP parity (TC-H12). See [ADR-019](../../adr/ADR-019-http-first-user-test-automation.md).

## Links

- User test cases: `1.places-agent/agent-specs/8.user-test-cases.md`
- Deployment § MCP: `1.places-agent/agent-specs/6.agent-deployment.md`
- Dual transport: [ADR-003](../../adr/ADR-003-dual-transport.md)
