---
title: HTTP clients read trip facts via fetch_trip_details only
type: ops-lesson
status: active
as_of: 2026-09-03
tags:
  - fetch_trip_details
  - travel_tips
  - visa_requirement
  - adr-046
related_spec: 3.where2play/2play-specs/2play-design.md
related:
  - adr/ADR-046-trip-store-pg-memory-fetch.md
  - adr/ADR-045-iconic-places-unified-acquisition.md
---

# HTTP trip reads are fetch-only

Any **HTTP** client of places-agent (where2play BFF and other apps) must obtain trip facts from `POST /v1/fetch_trip_details` (`trip_id` + `fields[]` + optional `day_index`). Write tools (`travel_tips`, `visa_requirement`, `discover_places`, `make_itinerary`, `plan_next_stop`) persist and may return `trip_id` / `revision` / progress. Internal `patchTrip` (ADR-049; replaces `patch_skeleton`) is **not** an HTTP/MCP tool. Their JSON/NDJSON bodies are **not** the product truth for skeleton, candidates, constraints, filled stops, or artifacts.

where2play Plan / saved itinerary surfaces render from those fetch slices. BFF may call write tools **once to persist**, then fetch.

**In scope for this rule:** must-see chips (fetch **`candidates`**), travel-tips four cards (fetch **`artifacts`** after make), visa, skeleton preview (including in-thread cards), filled stops, day themes, trip constraints shown on Plan.

**Out of scope:** POI encyclopedia facts via `get_place_details` when not already on the trip; auth/profile copy; i18n catalog strings (labels, not destination essays).

**Not this rule:** Cursor/ChatBox MCP hosts may still inspect write-tool envelopes. HTTP callers are not exempt.

`tips-prose` LLM, if it still runs on the **write** path, stores fields into `artifacts.tips`. Timeout: HTTP 200 when `iconic_places` exists; UI still fetch. Visa copy comes from Orizn adapter into `artifacts.visa`, then fetch — 2play must not treat `POST /v1/visa_requirement` as the render payload.

**Store merge:** `commitPatch` must **merge** nested `artifacts` (tips vs visa). A later tips write must not replace the whole `artifacts` object and drop visa.

**Revision after fetch:** after `make_itinerary`, 2play should send the **fetch** slice `revision` on the first `plan_next_stop`, not only the write envelope, when they differ.
