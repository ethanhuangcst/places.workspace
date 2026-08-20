---
title: Chat tools use ADR-026 provider auto-select
type: ops-lesson
status: active
as_of: 2026-08-20
tags:
  - what2eat
  - places-agent
  - chat
  - providers
related_spec: workspace-specs/adr/ADR-026-region-based-provider-auto-selection.md
related:
  - adr/ADR-026-region-based-provider-auto-selection.md
  - knowledge/ops/mvp3a-provider-auto-selection.md
---

# Chat provider auto-select (same as Decide)

## Summary

Decide HTTP search omits `providers[]` so places-agent picks AMAP/Google by destination. Chat previously let the LLM pass `providers: ["GOOGLE_MAPS"]`, which skipped auto-select. Chat now strips `providers` on search/geocode tool calls and uses `address ?? query` for region detection.

## Guidance

1. `omitChatToolProviders` in `src/agent/loop.ts` — chat path only; HTTP callers can still override.
2. `applyProviderStrategy` location hint: `input.address ?? input.query`.
3. Short street geocodes (e.g. fixture `吴中路`) fall through to coords; Latin formatted addresses without country markers stay `other` (not China’s bbox).
4. **Empty AMAP → Google once** when providers were auto-selected (`shouldTryGoogleAfterEmptyAmap`, [ADR-031](../../adr/ADR-031-amap-empty-google-fallback.md)). Explicit `providers: ["AMAP"]` does not fall back. Fixes chat misses like typo `吴记鲜定位` vs real `吴记鲜定味` when Decide/chat pin forces mainland AMAP.
5. Restart places-agent after deploy so prompt + loop changes load.

## Links

- [ADR-026](../../adr/ADR-026-region-based-provider-auto-selection.md)
- [ADR-031](../../adr/ADR-031-amap-empty-google-fallback.md)
- [MVP-4 follow-ups](./what2eat-mvp4-followups.md)
