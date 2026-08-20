# ADR-029: Decide criteria draft across locale refresh

## Status

Accepted

## Context

On Decide, switching header locale calls `persistLocale` then `router.refresh()`. That remounts or re-runs effects that hydrate the search form from profile (`GET /api/profile/personal`) and/or `SearchCache` criteria.

The profile effect historically did `setLocation(p.defaultLocation)` whenever URL had no `location`, **ignoring** in-progress edits (`locationTouched`). Users saw the area field snap back to the profile default after every locale switch. The same class of bug can hit meal context, budget, and craving if those fields are re-seeded the same way.

Alternatives considered:

1. **Only `locationTouched` in memory** — fails when the client remounts and state resets.
2. **Write drafts into the user profile** — conflates “search this time” with “my default address”; wrong product semantics.
3. **Encode drafts in the URL on every keystroke** — noisy history; conflicts with History re-run query params.
4. **sessionStorage draft + hydrate order** — survives `router.refresh()` for the tab session; does not change profile defaults.

## Decision

Persist Decide **criteria drafts** in **sessionStorage** (tab session). Hydrate in this order:

1. URL query (`location` / `meal` / `budget` / …) — History re-run wins  
2. sessionStorage draft keys (`w2e.decide.draft.*`)  
3. Latest non-expired SearchCache criteria (when present)  
4. Profile defaults — **only when the field is still virgin** (no draft, not touched)

Profile hydrate must never overwrite a touched field or an existing draft. Editing Decide does **not** update Profile defaults.

Canonical story: `decide-10` (location) and `decide-11` (meal / budget / craving) in what2eat MVP-4.

## Rationale

Matches user expectation (“keep what I just typed”). sessionStorage is enough for locale switches without promoting drafts to account settings. URL remains the override for History.

## Consequences

- Locale switch no longer resets Decide form to profile defaults.  
- Closing the tab clears drafts (acceptable).  
- Cross-tab sync is out of scope.  
- Implementers must gate profile effects with touched/draft checks, not only `searchParams`.

## Date

2026-08-20
