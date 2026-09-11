# ADR-064 — Takeoff submit confirm; origin sheet before confirm

- **Status:** Accepted (2026-09-10). Confirm body = **Option A** (summary).
- **Date:** 2026-09-10
- **Deciders:** product
- **Mock:** [`../2play-specs/ui-mockup/06-plan-takeoff-11.html`](../2play-specs/ui-mockup/06-plan-takeoff-11.html) (`?state=confirm` · `origin_then_confirm`)
- **Related:** [ADR-061](./ADR-061-takeoff-11-fields-skeleton-first.md) (takeoff 11 + origin overlay)
- **Stories:** `2play-plan-100` US6 · dest label AC2 examples · `agent-geocode-100` AC3/AC4

## Context

Takeoff must not native-submit on Enter (or CTA) without an explicit confirm sheet (same chrome as origin / hotel overlay). Ambiguous origin (e.g. `凯悦` in Lisbon) already opens the origin candidates sheet. Opening submit confirm at the same time stacks two dialogs.

## Decision

1. **Enter and「规划行程」** both enter the same submit pipeline (no direct plan start without confirm).
2. **One sheet at a time — origin first, then confirm:**
   - Validate fields → if origin non-empty, resolve origin.
   - `candidates` / `not_found` → **only** origin overlay; confirm stays closed.
   - Unique hit, skip, or empty origin → open submit confirm.
   - After origin pick or skip → close origin sheet, **then** open submit confirm.
   - Origin retry → stay on takeoff; no confirm.
3. **Destination verified label** stays under the input; input text is not rewritten (`里斯本` stays `里斯本`).
4. **Confirm body = Option A:** title + body + trip summary `dl` (destination label, dates/days, type/party, budget/pace/transit, origin/start time). **Option B** (title+body only) rejected after mock compare.

## Consequences

- Implementation must serialize overlays; never mount origin + confirm together.
- i18n keys for confirm title/body/actions and summary row labels (EN/CN/HK/TW).
- Live geocode: AMAP may omit `country` on mainland hits — default `中国` when mainland admin present; strip trailing `市` for city display (`杭州市` → `杭州`).

## Alternatives considered

| Option | Rejected because |
| --- | --- |
| Confirm first, then origin on OK | Double friction after user already confirmed |
| Combined sheet (candidates + confirm) | Breaks origin chrome; crowded |
| Stacked overlays | Ambiguous focus; confirm before hotel settled |
| Confirm only on Enter (CTA direct) | Rejected by product — both paths use confirm |
| Option B (copy only) | Product chose A after mock compare |
