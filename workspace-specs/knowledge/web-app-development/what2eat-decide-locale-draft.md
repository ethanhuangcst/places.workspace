---
title: Decide form draft vs locale refresh
type: ops-lesson
status: active
as_of: 2026-08-20
tags:
  - what2eat
  - i18n
  - decide
  - hydrate
related_spec: 2.what2eat/2eat-specs/2eat-stories.md
related:
  - adr/ADR-029-decide-criteria-draft-hydrate.md
  - knowledge/web-app-development/what2eat-mvp3-lessons.md
---

# Decide form draft vs locale refresh

## Summary

Header locale switch (`persistLocale` + `router.refresh()`) can re-run Decide profile hydrate and overwrite the area (and other criteria) with `defaultLocation`. Treat search-form values as a **session draft**, not as profile state.

## Evidence

- Profile effect in `decide-page.tsx` previously: if no URL `location`, always `setLocation(p.defaultLocation)`.
- `locationTouched` existed but was not consulted by that effect.
- Remount / new `searchParams` identity after refresh resets React state and re-triggers hydrate.

## Lesson / guidance

1. Hydrate order: URL → `sessionStorage` draft → SearchCache criteria → profile default (virgin only).  
2. On every edit of location / meal / budget / craving, write the draft key.  
3. Never use profile personal as a “reset on refresh” source after the user has edited.  
4. Do not persist Decide drafts into Profile unless the user saves on the Profile page.  
5. **SSR:** Never seed `useState` from `sessionStorage` / `localStorage`. First paint must use fixed defaults (`decideFormSsrDefaults` / `defaultChatPanelSize`); apply storage in a mount `useEffect`. Reading storage in the initializer causes hydration mismatch on chat context (and panel size).

## Chat blocks shape

LLMs often emit nested `{"heading":{...}}` when the prompt says `heading{level,text}`. Parser accepts both that shape and `{type:"heading",...}`; system prompt now shows explicit `type` examples.

## Links

- [ADR-029](../../adr/ADR-029-decide-criteria-draft-hydrate.md)  
- Stories `decide-10` / `decide-11` (MVP-4)
