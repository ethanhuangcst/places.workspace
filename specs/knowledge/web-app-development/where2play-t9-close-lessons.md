# MVP-T9 close lessons (chat 改行程 → ADR-071 descope)

**Date:** 2026-09-18 (descope) · **Verify note:** 2026-09-20 (quota restored)

## Delivered then removed (ADR-071)

## Delivered then removed (ADR-071)

T9 shipped refine briefly, then **descoped** to Replan-only:

1. ~~**agent-chat-93e** — `plan_trip` refine~~ **Removed**
2. **2play-plan-050** — BFF product LLM removed — **Kept**
3. ~~**2play-plan-90e** — `/api/chat` refine~~ **Removed**

**Current product:** After plan complete — no composer, no refine thread. Change trip via **Replan** only. **need_input** composer during planning unchanged.

## Verification (2026-09-20)

| Gate | Result |
| --- | --- |
| Vitest (thread, i18n, hydrate, no post-complete `plan-nav-input`) | Pass |
| Map quota (AMAP + Google) | **Restored** (2026-09-20) |
| Live browser (takeoff → plan → complete → Replan) | Product **usable Confirmed** 2026-09-20 |

DoD usability **Done**（2026-09-20）。

## Historical — refine regression (2026-09-18, pre-descope)

**Symptom:** Shanghai refine「第三天不要去天文馆…」→ false success; transit/meals stripped.

**Cause:** BFF skeleton merge; unstable agent refine.

**Outcome:** Product chose full descope (ADR-071) over ADR-070 skeleton refine loop.

## Related

- [ADR-071](../../adr/ADR-071-descope-in-page-refine-replan-only.md)
- [plan-trip-refine-t9.md](../agent/plan-trip-refine-t9.md) (Cancelled — historical)
- [shanghai-family-refine-probe.md](../agent/shanghai-family-refine-probe.md)
