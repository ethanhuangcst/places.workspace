---
title: HTTP clients read trip facts via fetch_trip_details only
type: ops-lesson
status: active
as_of: 2026-09-06
tags:
  - fetch_trip_details
  - travel_tips
  - visa_requirement
  - plan_trip
  - adr-046
  - adr-050
  - adr-051
  - adr-053
related_spec: 3.where2play/2play-specs/2play-design.md
related:
  - adr/ADR-046-trip-store-pg-memory-fetch.md
  - adr/ADR-045-iconic-places-unified-acquisition.md
  - adr/ADR-050-where2play-no-product-llm.md
  - adr/ADR-051-discover-resolve-display-photo.md
  - adr/ADR-053-origin-stay-as-stop-card.md
  - ../../1.places-agent/agent-specs/real-agent-refactory.md
---

# HTTP trip reads are fetch-only

Any **HTTP** client of places-agent (where2play BFF and other apps) must obtain trip facts from `POST /v1/fetch_trip_details` (`trip_id` + `fields[]` + optional `day_index`). Write tools (`plan_trip` Target；as-built 仍可能 `travel_tips` / `visa_requirement` / `discover_places` / `make_itinerary` / `plan_next_stop`) persist and may return `trip_id` / `revision` / progress. Internal `commit_trip` / `patchTrip` is **not** an HTTP/MCP tool. Their JSON/NDJSON bodies are **not** the product truth for skeleton, candidates, constraints, filled stops, or artifacts.

where2play Plan / saved itinerary surfaces render from those fetch slices. BFF may call write tools **once to persist**, then fetch.

**In scope for this rule:** must-see chips (fetch **`candidates`**), attraction / meal / **origin stay** thumbnails and pins (`photos[0]` / `native_id` / coords from the **same** card — ADR-051/053；`stop-origin` 只读已抄的 stay 卡), travel-tips four cards (fetch **`artifacts`**), visa, skeleton preview, filled stops, day themes, trip constraints (`originStay`) shown on Plan. HTTP clients must not resolve Google/Amap photos or re-search an ingested origin by name. List rows render the slot card only（ADR-052 D9）；place-sheet details use `slot.provider` + `locale`（D10）— not a second vendor fan-out.

**Out of scope:** POI encyclopedia facts via `get_place_details` when not already on the trip; auth/profile copy; i18n catalog strings (labels, not destination essays).

**Not this rule:** Cursor/ChatBox MCP hosts may still inspect write-tool envelopes. HTTP callers are not exempt.

**where2play 零产品 LLM（ADR-050 Target）：** BFF **不得**用本地模型生成介绍/着装/安全/必去名单。`tips-prose` 仅在 agent 写路径；结果进 `artifacts.tips`，UI 仍 fetch。Visa → `artifacts.visa` → fetch — 不把 `POST /v1/visa_requirement` 当渲染载荷。

**Store merge:** `commitPatch` must **merge** nested `artifacts` (tips vs visa). A later tips write must not replace the whole `artifacts` object and drop visa.

**Revision after fetch:** after a write that creates/updates skeleton，2play should send the **fetch** slice `revision` on the next write（Target: 后续 `plan_trip`；as-built: first `plan_next_stop`），not only the write envelope, when they differ.
