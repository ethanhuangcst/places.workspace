# UI copy: 框架 vs skeleton（骨架）

- **Date:** 2026-09-10
- **Products:** where2play Plan / 行程助手
- **Binding:** [`2play-design.md`](../../2play-specs/2play-design.md) §4.7.1 · [`2play-plan-101`](../../2play-specs/2play-stories.md)

## Rule

| Layer | Term |
| --- | --- |
| **User-visible UI / i18n / mock source copy** | **框架**（行程框架、生成框架、框架就绪…） |
| **Internal specs / code / ADR / CSS / APIs** | `skeleton` / 骨架（make/commit skeleton, `.slot--skeleton`, fetch `skeleton` field） |

**Do not** put「骨架」in product strings, assistant bubbles, phase bars, or dialogs. Prefer「框架」in CN/HK/TW catalogs; EN should use “itinerary outline” / “trip framework” (or existing key semantics), not the word “skeleton” in user-facing copy.

## Assistant chat chrome

User-facing agent **notice** sentences use `.bubble.bubble--agent.bubble--agent-notice`: white chat bubble (rounded with small bottom-left corner, light hairline), paired with user peach `#ffe3d3` bubbles. Progress / route-spine stay transparent in the thread.

## Why

「骨架」reads as engineering jargon. Travelers understand「行程框架」; engineers keep `skeleton` in contracts.
