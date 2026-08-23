---
title: agent-mate.ai operator visual family
type: design-direction
status: active
as_of: 2026-08-19
tags:
  - ui
  - admin
  - places-agent
  - kb-agent
related_spec: 1.places-agent/agent-specs/3.ui-design.md
related:
  - knowledge/handbook.md
  - knowledge/i18n/hk-tw-output.md
  - adr/ADR-011-hk-tw-independent-locales.md
  - adr/ADR-012-admin-ui-on-agent.md
---

# Operator UI — kb 性冷淡 family

## Summary

places.agent-mate.ai operator chrome matches [kb.agent-mate.ai](https://kb.agent-mate.ai): `#fafafa` field, black ink, hairline rules, radius 0. Personality is only `agent-logo.png`. Pixel contract: [`3.ui-design.md`](../../../1.places-agent/agent-specs/3.ui-design.md). Clickable mock-ups: [`ui-mockup/`](../../../1.places-agent/agent-specs/ui-mockup/).

## Evidence

- User brief: 性冷淡, screenshots of kb home / guide / login / users.
- Live kb tokens (2026-08-17): `--bg #fafafa`, Outfit + Noto Sans + JetBrains Mono, underline fields, black rectangle buttons.
- kb locale switcher is `中文` / `EN` (two locales). places-agent has four product ids (`EN` `CN` `HK` `TW`, ADR-011).
- kb Users list can “View” a full key again. places-agent AC: plaintext only at create / regenerate (ADR-012).
- Black-background logo on `#fafafa` read as a black tile. Replacing `(0,0,0)` via flood-fill from corners also ate sketchy black outlines (outlines are the same black and connected to the field).

## Lesson / guidance

- **Copy kb structure and tokens**, not kb product copy. Default landing after login is **Caller API keys**, not kb’s Users list.
- **Locale switcher** shows the four protocol ids `EN CN HK TW` (not localized, not a 中文/EN pair). `HK` vs `TW` catalogs must differ. Admin chrome 書面語 vs 口語: follow [`i18n/hk-tw-output.md`](../i18n/hk-tw-output.md) (書面語 for chrome). Mock-up HK strings that use 畀/嘅 are dialect sketches, not the production chrome rule.
- **Logo** sits on `--bg` `#fafafa` (same as the page). To swap a black field: keep pixels with chroma (or near colored blobs via dilate), then recolor only the remaining dark field. Regenerated favicon / apple-touch from that PNG.
- **Home tagline** leads with “Places agent.” (EN) / 地点智能体 / 地點智能體. Closed-register notice stays **one line** at desktop (`nowrap` + slightly wider login card); wrap only on very narrow viewports.
- **Static mock-ups:** inline locale catalogs in `i18n.js`. `fetch()` of JSON fails on `file://`.
- Do not put map-vendor keys, Portainer, or OPENAI_CN in this UI.

### Integration guide (`/instructions`)

- Content width `--guide-max: 56rem`. Same URL signed-in or not; signed-in visitors keep app chrome.
- **Agent capabilities** is first. Table columns: Capabilities · Channel · Description. Do not keep a separate Tool column. Architecture does not repeat the tool list.
- First six Capabilities cells are tool-name literals (`search_restaurants` … `plan_itinerary`). Last two: i18n Place chat and literal `Tripadvisor.enrich`. Channel: `HTTP and MCP` vs `HTTP only` (Place chat also shows `POST /v1/chat`).
- **Hairlines:** TOC has no bottom rule. First section heading has no top rule. Last table row has no bottom rule so it does not stack with the next `h2` top rule — keep **one** section divider.
- Integration guide links (`admin.home.instructions_link`, `admin.landing.instructions_link`) open a new tab. E2E uses `page.expect_popup()`.

## Links

- Spec: [`3.ui-design.md`](../../../1.places-agent/agent-specs/3.ui-design.md)
- [ADR-012](../../adr/ADR-012-admin-ui-on-agent.md) — admin UI on the places-agent deployable
- [ADR-011](../../adr/ADR-011-hk-tw-independent-locales.md) — four locales, no OpenCC
- [ADR-020](../../adr/ADR-020-http-only-chat-and-enrich.md) — chat and enrich HTTP-only
