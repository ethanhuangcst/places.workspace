# ADR-011: Independent zh-HK and zh-TW output (no conversion)

## Status
Accepted

## Context

places-agent must emit user-facing text in `EN`, `CN` (`zh-CN`), `HK` (`zh-Hant-HK`), and `TW` (`zh-Hant-TW`). Hong Kong and Taiwan both use Traditional Chinese, but written vocabulary, place names, and number/date formatting differ. Naive options were: one Traditional catalog plus OpenCC; reuse TW copy for HK (or vice versa); let the LLM “just write Cantonese/Taiwanese”; or maintain two catalogs plus vendor `language` plus formatters.

## Decision

1. Treat **`HK` and `TW` as independent locales**. Never use OpenCC (or similar) to convert TW copy into HK copy, or the reverse.
2. Split output into **three layers** and localize each layer with the right mechanism:
   - **A — Agent copy:** i18n catalogs (`zh-HK` vs `zh-TW`). Missing key → `EN` → key. Not HK↔TW.
   - **B — Vendor facts:** request Google (and similar) with `languageCode` `zh-HK` / `zh-TW` / `zh-CN` / `en`. Do not rewrite official names with a glossary.
   - **C — Formatters:** `Intl` (or equivalent) with `zh-HK` vs `zh-TW`. Currency from place facts (HKD vs TWD), not from UI locale alone.
3. **LLM prose** (NL chat, itinerary narrative) uses a **versioned locale instruction plus a small travel glossary**, loaded only when `locale` is `HK` or `TW`. HK written output is **書面語** with HK lexicon, not Cantonese 口語 (no 唔該/喺 unless the user wrote that way and we echo).
4. Tools stay locale-agnostic; locale is an **input on the request** and an **output contract**, not a separate tool per language.
5. The travel glossary is **not exhaustive**. Unseen regional words are handled by vendor `languageCode` (facts) and the LLM’s locale instruction (prose). Locale is **never** inferred by classifying result tokens. New systematic pairs are added to catalogs after eval misses.

## Rationale

- OpenCC and character-set converters do not map 的士↔計程車 or 港鐵↔捷運.
- Sharing one Traditional catalog produces the wrong city dialect and trains reviewers to ignore locale bugs.
- Vendor names are source-of-truth; a glossary rewrite invents names Google/AMAP did not return.
- Full Cantonese 口語 is hard to QA and mismatches typical HK product UI (Apps, MTR, government).
- Agent-builder: glossary is on-demand knowledge, not front-loaded every turn.

Alternatives rejected: single `zh-Hant` catalog; HK fallback to TW; post-hoc LLM “translate this JSON to Cantonese”.

## Consequences

- Four catalog files from the first UI/agent strings (not “TW later”).
- Google Places/Geocoding/Routes calls must pass `languageCode` from the request locale map.
- Prompt + glossary are versioned with the model/Quanzil config (ADR-004); rollback = pin prompt version.
- Tests assert keys and locale codes, plus a small glossary fixture — not English-only copy as the contract.
- Follow-up: Feature 13 AC already names four locales; implementation must not collapse HK/TW.

## Date
2026-08-17
