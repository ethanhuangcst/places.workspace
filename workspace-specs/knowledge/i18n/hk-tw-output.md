---
title: HK vs TW output (three layers)
type: research-note
status: active
as_of: 2026-08-17
tags:
  - i18n
  - zh-HK
  - zh-TW
  - places-agent
related_spec: 1.places-agent/agent-specs/1.agent-stories.md
related:
  - knowledge/handbook.md
  - knowledge/agent/places-agent-loop.md
  - adr/ADR-011-hk-tw-independent-locales.md
  - adr/ADR-004-quanzil-fixed-per-deployable.md
  - adr/ADR-014-open-meteo-weather.md
---

# HK vs TW — how places-agent should emit text

## Summary

Hong Kong and Taiwan both use Traditional Chinese, but they are **different locales**. Do not convert catalogs with OpenCC. Localize **agent copy**, **vendor facts**, and **numbers** with three different mechanisms. LLM narrative gets a **small glossary on demand**, not a full language model swap.

## Evidence

- Product locales: `EN`, `CN` (`zh-CN`), `HK` (`zh-HK` / `zh-Hant-HK`), `TW` (`zh-TW` / `zh-Hant-TW`) — Feature 13 / Feature 19.
- Google Places/Geocoding: `languageCode` `zh-HK` vs `zh-TW` returns different official strings (e.g. 中環 vs 中正區-style TW names; HK MTR vs TW 捷運 in addresses when the vendor has them).
- OpenCC converts 简繁 character sets; it does **not** map regional lexicon (的士/計程車).
- HK consumer apps and government sites use **書面語** (您/請/的士) rather than 口語 (你/唔該/喺) for chrome UI.

## Lesson / guidance

### Locale map (request → catalogs + vendors + Intl)

| Product locale | BCP-47 / catalogs | Google `languageCode` | `Intl` locale |
| --- | --- | --- | --- |
| `EN` | `en` | `en` | `en` |
| `CN` | `zh-CN` | `zh-CN` | `zh-CN` |
| `HK` | `zh-HK` | `zh-HK` | `zh-HK` |
| `TW` | `zh-TW` | `zh-TW` | `zh-TW` |

Fallback for **missing catalog keys:** requested locale → `EN` → raw key. **Never** HK→TW or TW→HK.

### Layer A — Agent copy (deterministic)

Errors, HTTP/MCP wrappers, itinerary labels, admin chrome: **ICU / JSON catalogs**, one file per locale. Same **key**, different **value**.

### Layer B — Vendor facts (do not rewrite)

Place `name`, `formatted_address`, editorial snippets from Google/AMAP/Tripadvisor: pass `languageCode` / AMAP `language`. If the vendor returns English only, show English + provenance; do **not** glossary-patch “Victoria Harbour” → “維多利亞港” unless a second vendor returned that string.

**Exception — weather:** Open-Meteo is **not** a place vendor and has no `language` parameter. Forecast labels in their docs are English; the API returns WMO `weather_code`. Treat weather as **Layer A** (`weather.wmo.{code}`), not Layer B pass-through ([ADR-014](../../adr/ADR-014-open-meteo-weather.md)).

### Layer C — Formatters

Distance, duration, dates, times: `Intl.NumberFormat` / `DateTimeFormat` with the table above. **Currency code** comes from place/country facts (`HKD` vs `TWD` vs `CNY`), not from “user picked HK UI so everything is HKD”.

### Layer L — LLM prose (constrained)

NL chat replies and itinerary *narrative* sentences: system instruction includes locale + glossary **only when locale is HK or TW**. Keep tools and JSON fields in English keys; translate only user-visible strings. Weather narrative must use catalog condition text for the request locale — do not leave Open-Meteo English phrases in `CN` / `HK` / `TW` output.

**HK instruction (written, not 口語):** Traditional Chinese, Hong Kong written standard, use glossary terms, do not use Taiwan 捷運/計程車 for HK streets, do not use 简体.

**TW instruction:** Traditional Chinese, Taiwan written standard, inverse glossary.

### Travel glossary (agent copy + LLM; not vendor names)

| Sense | HK (`zh-HK`) | TW (`zh-TW`) | CN (`zh-CN`) | EN |
| --- | --- | --- | --- | --- |
| Taxi | 的士 | 計程車 | 出租车 | taxi |
| Metro | 港鐵 | 捷運 | 地铁 | MTR / metro |
| Bus | 巴士 | 公車 | 公交车 | bus |
| Tram | 電車 | 路面電車 | 有轨电车 | tram |
| Info / kiosk | 諮詢處 | 服務台 | 问讯处 | information |
| Software | 軟件 | 軟體 | 软件 | software |
| Quality | 質素 | 品質 | 质量 | quality |
| Taxi stand | 的士站 | 計程車招呼站 | 出租车站 | taxi stand |
| Light rain showers (WMO 80) | 微驟雨 | 小陣雨 | 小阵雨 | slight rain showers |
| Thunderstorm (WMO 95) | 雷暴 | 雷雨 | 雷暴 | thunderstorm |

Extend this table in catalogs (`nav.mode.taxi`, `weather.wmo.80`, etc.), not as a one-off prompt dump that grows unbounded.

### Open-Meteo weather codes (Layer A)

Open-Meteo `weather_code` is a WMO integer. Map every code used in the product to `weather.wmo.{code}` in all four catalogs. Do not display the English Open-Meteo documentation string unless locale is `EN`.

### Open vocabulary (words not in the glossary)

The glossary is a **seed for QA**, not a closed dictionary. places-agent does **not** classify result text to decide HK vs TW.

| What varies | How it is chosen | What happens for unseen words |
| --- | --- | --- |
| UI locale (`HK` vs `TW`) | Request field (`locale` / `Accept-Language` from the caller). Never inferred from POI strings. | Unlisted chrome strings: add a catalog key when we ship that UI. |
| Place geography (Hong Kong vs Taiwan) | Search coords / `country` on the place. Independent of UI locale. | Currency and “which metro brand” follow **place country**, not the glossary. |
| Vendor names/addresses | Google/AMAP `languageCode` from UI locale. | Vendor lexicon is their dataset. Pass through + provenance. Do not match against our table. |
| Weather condition | Open-Meteo `weather_code` → `weather.wmo.{code}` for the request locale. | Unknown code: `EN` then the key. Add the WMO row to catalogs; do not show English docs in CN/HK/TW. |
| LLM narrative | Locale instruction + small glossary. | Model already knows many HK vs TW pairs (軟件/軟體, 網絡/網路). Glossary pins travel terms we must not mix. If eval shows a repeated miss, **add that pair** to the catalog/glossary. |

Do **not** scan results for 的士 vs 計程車 to flip locale. That mixes destination facts with UI language and fails on new words by design.

### What not to do

- OpenCC or `zh-Hant` collapse of HK and TW
- Translating JSON then hoping the model “sounds local”
- Glossary-rewriting official POI names
- Serving 口語 Cantonese as default HK UI
- Showing Open-Meteo English weather phrases in `CN` / `HK` / `TW` (translate `weather_code`)

## Links

- [ADR-011](../../adr/ADR-011-hk-tw-independent-locales.md)
- [ADR-014](../../adr/ADR-014-open-meteo-weather.md)
- Agent Feature 13: `1.places-agent/agent-specs/1.agent-stories.md`
- App Feature 19: same file, admin i18n
