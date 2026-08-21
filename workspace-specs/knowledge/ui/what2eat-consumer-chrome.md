---
title: what2eat consumer chrome — nav order and Profile CJK
type: design-direction
status: active
as_of: 2026-08-21
tags:
  - ui
  - what2eat
  - i18n
  - header
related_spec: 2.what2eat/2eat-specs/2eat-design.md
related:
  - knowledge/handbook.md
  - knowledge/ui/agent-mate-admin-visual.md
  - knowledge/i18n/hk-tw-output.md
  - adr/ADR-011-hk-tw-independent-locales.md
---

# what2eat consumer chrome — nav order and Profile CJK

## Summary

Signed-in header nav order is **Decide → Saved → Profile** (not Decide → Profile → Saved). History is not a fourth top-level item; on `/history`, **Saved** stays `is-active`. Profile CJK nav label is **用户档** (CN) / **用戶檔** (HK, TW), not「定口味」.

## Evidence

- Product ask (2026-08-21): reorder header to Decide / Saved / Profile.
- Product ask (2026-08-21): replace「定口味」with「用户档」for Profile nav.
- Implemented in `2.what2eat/src/ui/app-header.tsx`, `messages/{CN,HK,TW}.json`, mock `ui-mockup/` + `i18n.js`, and `2eat-stories.md` `header-01`.
- Operator chrome (places-agent) remains a separate 性冷淡 family — do not copy consumer nav labels there.

## Lesson / guidance

1. **Nav DOM order is product contract** — keep mock HTML, Next `AppHeader`, and `header-01` AC in the same Decide → Saved → Profile sequence.
2. **History inherits Saved active** — `pathname` starts with `/history` must mark Saved, not leave all items inactive.
3. **Profile CJK is account chrome, not taste chrome** —「用户档 / 用戶檔」names the section; taste chips stay inside the page. Do not revive「定口味」on the nav.
4. **HK ≠ TW catalogs** still apply (ADR-011); this label happens to share the same traditional form in both.

## Links

- Spec: [`2eat-design.md` §1.3](../../../2.what2eat/2eat-specs/2eat-design.md)
- Stories: [`2eat-stories.md` header-01](../../../2.what2eat/2eat-specs/2eat-stories.md)
- Operator contrast: [`agent-mate-admin-visual.md`](./agent-mate-admin-visual.md)
