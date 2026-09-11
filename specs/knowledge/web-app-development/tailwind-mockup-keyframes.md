# Knowledge — Tailwind v4 + large mockup.css keyframes

- **Date:** 2026-09-10
- **Area:** where2play CSS / Turbopack

## Lesson

Appending a large `@keyframes` block (especially with multi-stop selectors like `0%, 100%` and `color-mix` box-shadows) to `app/mockup.css` imported from `globals.css` via `@import "tailwindcss"` can make **Turbopack / `@tailwindcss/postcss`** report a misleading `Missing closing } at .plan-nav__rail` even when brace counts are balanced.

A leaner `.plan-progress*` block **without** `@keyframes plan-progress-pulse` processed cleanly; `npx`/postcss CLI and Next agreed after `.next` cache clear.

## Practice

1. Prefer minimal progress styles for product CSS; keep ornate keyframes in the static mock gallery if needed.
2. After CSS appends that break the app, delete `.next` before re-testing — Turbopack can keep serving a prior fatal CSS error.
3. Validate with `postcss([tailwind]).process(globals.css)` before relying on `next dev`.
