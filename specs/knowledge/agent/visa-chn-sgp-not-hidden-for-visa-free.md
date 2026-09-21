---
title: Visa CHN→SGP missing link — not because visa-free
type: research-note
status: active
as_of: 2026-09-21
tags:
  - visa
  - orizn
  - singapore
related_spec: specs/2play-specs/2play-stories.md
related:
  - adr/ADR-044-orizn-visa-rest-adapter.md
  - adr/ADR-042-no-city-encyclopedia-in-source.md
---

# Visa CHN→SGP missing link — not because visa-free

## Summary

A China-passport Singapore plan that shows no VISA link is **not** explained by Orizn returning visa-free. Live Orizn (2026-09-21) returns `requirement: "visa_free"`, `visa_free_days: 30`. Product display already maps that to a shown popover title. Hide/empty comes from other paths (dest country missing, 94c notice, hydrate gap, race).

## Evidence

### Orizn live (MCP `check_visa_requirement`, lang=zh)

| Pair | requirement | Notes |
| --- | --- | --- |
| CHN→SGP | `visa_free` / 30 days | documents: passport, return ticket, lodging, funds; process: no prior visa, stamp on arrival. **No** `arrival_card` / 入境卡 / SGAC field. |
| CHN→CHN | `not_applicable` | Home country; no stay limit. |
| CHN→HKG | `special` | Exit-entry Permit + endorsement — not visa-free. |
| CHN→MAC | `visa_required` | Generic embassy template; `verified: false`. |
| CHN→TWN | `visa_required` | Same generic template; `verified: false`. |

Agent fixture `CHN:SGP` matches visa_free 30 days.

### Product (as of 2026-09-21)

- `visaTipsDisplay` maps `visa_free` → i18n 免签（最长 N 天）. Does not return null for visa_free.
- `artifactsVisaFromSlice` drops only `unavailable` / `errors.*` / bad alpha-3.
- 94c hides or notices for missing dest country, missing nationality, or Orizn fail — **not** for visa_free.

### Likely causes when Singapore showed no VISA (check in order)

1. Geocode omitted `country_code` → no `destinationCountryAlpha3` → skip Orizn + hide (94c).
2. 94c `visa_notice` (quota / unconfigured) — text without `.travel-tips-visa-link`.
3. `2play-plan-106` not done — leave Plan and return; `travelTips` not re-fetched from artifacts.
4. Race: tips poll before visa dualWrite; late fetch miss.

## Lesson / guidance

1. **Show visa-free.** Do not treat `visa_free` as empty.
2. **Hide only same ISO.** `passportAlpha3 === destinationCountryAlpha3` (Orizn `not_applicable`). Do **not** collapse 中国大陆/港澳台 into one domestic zone.
3. **No invented 入境卡.** Orizn CHN→SGP has no arrival-card field. ICA page may mention SG Arrival Card; product must not invent it (ADR-042 / 94c honesty). Map only vendor strings when present.

## Links

- Story follow-up: `2play-plan-107` (same-ISO hide) in `2play-stories.md`
- Hydrate gap: `2play-plan-106`
- ADR-044 Orizn adapter
