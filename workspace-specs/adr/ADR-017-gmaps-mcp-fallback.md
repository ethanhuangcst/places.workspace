# ADR-017: Google Maps Cloudflare Worker MCP as transport fallback

## Status

Accepted

## Context

Callers may request `GOOGLE_MAPS` (ADR-005). places-agent on 野草云3 (HK) can usually reach `maps.googleapis.com`. Direct Google REST can still fail (timeout, reset, unexpected CN egress). Alternatives: fail the vendor; silently switch to AMAP; use Google only via a Cloudflare Worker MCP; or try direct first and MCP second.

## Decision

1. For `GOOGLE_MAPS`, call **direct** Google Maps Platform REST first (`GOOGLE_MAPS_API_KEY` on places-agent).
2. On **egress failure** only, call the **Cloudflare Worker MCP** (`GMAPS_MCP_URL` + `GMAPS_MCP_BEARER`, Streamable HTTP). Discover tools with `tools/list`.
3. Provenance stays **`GOOGLE_MAPS`**. The Worker is not a fourth `providers[]` id and not an AMAP replacement.
4. The Worker holds the Maps Platform key. The agent sends only the MCP bearer — never the Maps key to the Worker, never the bearer to browsers.
5. Do **not** fall back to AMAP unless the caller asked for `AMAP`. Mainland **client** inability to open Google Maps remains a deeplink/UI concern (ADR-006).

## Rationale

- Direct REST is simpler and cheaper when HK egress works.
- MCP fallback preserves Google data when the agent cannot reach Google, without violating caller `providers[]`.
- Switching to AMAP on Google failure would reintroduce destination-forced vendors (ADR-005).

Rejected: MCP-only Google; AMAP as Google failure fallback; `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY`.

## Consequences

- Env needs both `GOOGLE_MAPS_*` and `GMAPS_MCP_*`. If MCP is unconfigured, skip Google with a skip reason after direct failure.
- Adapter tests: fixture direct success; fixture direct fail → MCP; never treat MCP as a separate provider in `sources[]`.
- Weather is unrelated: Google Weather API is not adopted (ADR-014).

## Date

2026-08-17
