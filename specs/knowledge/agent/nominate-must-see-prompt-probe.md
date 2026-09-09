# Must-see nominate prompt probe (2026-09-08)

**Scope:** prompt text only. No city POI tables (ADR-042).

Three user-message variants for `nominateMustSeeViaLlm` (`numDays` 4 Lisbon / 3 杭州):

| Id | Hint |
| --- | --- |
| A | Baseline: iconic names, JSON array only |
| B | A + reachable day-trip / famous outskirts when `numDays>=3` |
| C | B + one listing per landmark cluster (no ticket office / reconstruction hall) |

**Selected for product:** Direct-chat line (no JSON / theme-park lecture). Example:

`为以下行程目的地推荐必去地，仅列出地名，不需要安排行程： 杭州 3天 2人 情侣浪漫 豪华 轻松节奏 打车优先`

Params come from takeoff (city, days, party, type, budget, pace, transit). Parser accepts JSON or heading lists. Grounding remains `search_places`.

Human audit of live names: [`nominate-must-see-10city-audit.md`](./nominate-must-see-10city-audit.md).

Prompt revision draft (pending review): [`../../agent-specs/draft-nominate-must-see-prompt.md`](../../agent-specs/draft-nominate-must-see-prompt.md).

Live name lists are not stored here (would become an encyclopedia).
