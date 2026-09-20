---
title: tips-prose mixed-language from English weather drivers
type: ops-lesson
status: active
as_of: 2026-09-20
tags:
  - travel_tips
  - tips-prose
  - i18n
  - weather
  - adr-045
related_spec: ../../agent-specs/agent-stories.md
related:
  - adr/ADR-045-iconic-places-unified-acquisition.md
  - agent/iconic-display-travel-tips-only.md
---

# tips-prose 中英混排与 weather drivers

**Symptom:** CN locale 出行贴士卡 03 出现「备折叠伞防 drizzle。」（中英混排）。

**Root cause:** `travel_tips` tips-prose 把内部英文 `WeatherDriver` / severity 枚举（如 `drizzle`、`caution`）原样写入 LLM user message（旧 `weatherContext`）。模型在 `clothing` 里抄写了这些 token。where2play 只 fetch `artifacts.tips` 原样展示，不是 UI 翻译失败，也不是 locale 未传到 agent。

**Contrast:** `weather.summary` 已走 `t(locale, itinerary.weather.impact_*)`，卡 02 中文正确；仅 prose 分支上下文未本地化。

**Fix (agent):** `weatherContextForTips(locale, weather)` 只注入本地化 summary + `itinerary.weather.driver_*` 文案；CN/HK/TW 校验拒绝含英文 weather token 的 prose 并 retry 一次。2play 不客户端清洗。
