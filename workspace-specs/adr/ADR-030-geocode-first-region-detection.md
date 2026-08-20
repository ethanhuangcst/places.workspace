# ADR-030: Geocode-first region detection (replaces CJK heuristic)

## Status

Accepted — supersedes the CJK character ratio heuristic in ADR-026

## Context

ADR-026 introduced region-based provider auto-selection with three detection methods: marker lists, CJK character ratio >30%, and coordinate bounding boxes. The CJK heuristic caused repeated false positives:

- "臺北遠山" (Taiwan, traditional) → misclassified as mainland (missing from simplified marker list)
- "中環" (Hong Kong district) → misclassified as mainland (not in HK_MARKERS)
- "銀座" (Tokyo) → would be misclassified as mainland (100% CJK)
- "明洞" (Seoul) → same problem

Adding more markers is a whack-a-mole game. The fundamental issue: CJK text ≠ mainland China.

## Decision

1. **Google Geocode is the primary detection method** when the caller provides a text address without coordinates. The geocoded `formatted_address` contains the country (e.g., "China", "Hong Kong", "Japan") — parse it to determine region.

2. **Coordinate bounding boxes remain priority 1** (instant, no API call). Used when caller provides `near` coordinates.

3. **Marker lists become offline fallback** — used only when geocode fails (network error, timeout).

4. **CJK character ratio heuristic is deleted entirely.** Unknown CJK text defaults to "other" (Google), which can disambiguate itself.

5. **Geocode address text takes priority over geocoded coordinates** in region determination. China's bounding box (lat 18-54, lng 73-135) overlaps with Korea, Mongolia, and parts of Japan — the address text "South Korea" is unambiguous while coords are not.

6. **`resolveProviderStrategy` becomes async** to support the geocode call. It accepts an optional `geocodeFn` parameter for dependency injection (testability).

## Rationale

- Google Geocode correctly disambiguated every test case: "紫藤路128弄" → China, "中環" → Hong Kong, "銀座" → Japan, "春熙路" → Sichuan China
- The geocode call uses the existing `withGoogleTransport` fallback (direct → Cloudflare Worker MCP), so it works in mainland China
- Geocode latency is ~80ms and only triggered when no coordinates are provided (what2eat's normal flow already includes coords)
- Marker lists still work offline — graceful degradation, not hard dependency

**Rejected:** Expanding marker lists indefinitely (unmaintainable); using locale as primary signal (user in Shanghai searching Hong Kong would get wrong results); removing all text-based detection (coordinates not always available).

## Consequences

- `src/adapters/provider-resolver.ts` is now async; all callers use `await`
- `src/core/tools.ts` injects `buildGeocodeFn()` using Google live adapter
- `OVERSEAS_CJK_MARKERS` list deleted (no longer needed without CJK heuristic)
- Tests use mock `geocodeFn` for deterministic behavior

## Date

2026-08-20
