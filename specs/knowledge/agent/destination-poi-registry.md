# Destination POI registry (F87)

Family backlog: [`product-backlog.md`](../../product-backlog.md)

**As of:** 2026-09-04  
**Related:** ADR-049 D4–D6, ADR-042

Cross-trip **runtime** table (`Destination` + `AttractionPoi`), keyed by geocode `place_id` or `queryNorm` + rounded lat/lng — not a TypeScript city catalog.

Upsert only F84-eligible cards with `sources.native_id`. Restaurants never stored. Make merges registry then live search. L1 `getPlaceDetails` is `setImmediate` after upsert; missing details do not dirty trip candidates.

Apply migration `20260904000000_destination_poi_registry`. Tests use an in-memory store (`VITEST`). Restart `tsx` after core changes.

No new HTTP/MCP list tool. Itinerary truth remains Trip + fetch.
