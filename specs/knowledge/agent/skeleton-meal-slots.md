# Skeleton meal slots (F85)

Family backlog: [`product-backlog.md`](../../product-backlog.md)

**As of:** 2026-09-04  
**Related:** ADR-049, Feature 85

`make_itinerary` meal stops are **slots**, not venues. After parse, `name` is the slot id (`lunch` / `dinner` / `afternoon_tea`). Validation requires lunch every day and dinner when pace is medium or tight, even if the restaurant pool is empty.

UI must map `meal_slot` through i18n keys (`play.plan.meal_slot_*`). Do not render `name` as a shop. Restaurant search stays in F86 `plan_next_stop`.

`tsx` does not hot-load `src/core` — restart the agent after make-pipeline changes.
