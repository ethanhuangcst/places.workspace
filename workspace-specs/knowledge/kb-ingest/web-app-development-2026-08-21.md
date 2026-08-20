# KB ingest pack — what2eat MVP-4 close + follow-ups (2026-08-21)

Project: **places-workspace**. No secrets. Product AC stays in `2.what2eat/2eat-specs/`.

## Gaps proposed (2026-08-21)

| Item | Source | Why ingest |
| --- | --- | --- |
| ADR-029 | `adr/ADR-029-decide-criteria-draft-hydrate.md` | Decide draft hydrate order |
| ADR-030 | `adr/ADR-030-geocode-first-region-detection.md` | Geocode-first region (if not already in kb) |
| ADR-031 | `adr/ADR-031-amap-empty-google-fallback.md` | Empty AMAP → Google once |
| MVP-4 lessons | `knowledge/web-app-development/what2eat-mvp4-lessons.md` | Slice close notes |
| MVP-4 follow-ups | `knowledge/web-app-development/what2eat-mvp4-followups.md` | Timeout / providers / card-first map |
| Chat timeout | `knowledge/web-app-development/what2eat-chat-agent-timeout.md` | 25s ≠ agent down |
| Chat providers | `knowledge/web-app-development/what2eat-chat-provider-auto-select.md` | Strip providers + ADR-031 |
| Decide SSR draft | `knowledge/web-app-development/what2eat-decide-locale-draft.md` | SSR + draft rules |
| Agent loop refresh | `knowledge/agent/places-agent-loop.md` | ADR-030/031 + chat strip |
| Local daemon | `knowledge/ops/places-agent-local-daemon.md` | macOS agent shell + what2eat |

## Confirm

User requested propose + confirm in the same turn (`kb_import_documents` → `kb_confirm_import_batch` with `confirm_all_viable=true`).
