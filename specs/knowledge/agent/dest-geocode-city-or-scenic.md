---
title: Destination geocode must be city or scenic, not housing estate
type: ops-lesson
status: active
as_of: 2026-09-20
tags:
  - geocode
  - amap
  - takeoff
  - adr-042
related_spec: ../../agent-specs/agent-stories.md
related:
  - adr/ADR-042-no-city-encyclopedia-in-source.md
  - agent/origin-geocode-without-city.md
---

# Takeoff dest: reject 同名住宅区

**Symptom:** 起飞输入「鼓浪屿」显示「中国/西宁市」，后续搜不到景点。

**Root cause:** AMAP `/v3/geocode/geo` 把全国同名小区排在前面（`level=住宅区`，西宁 101.82,36.59）。adapter 盲取 `geocodes[0]`。厦门岛不在 geo 前 10 条；`/v5/place/text` 首条才是鼓浪屿风景名胜区 / 厦门市。

**Fix (`agent-geocode-114`):** geo 跳过住宅区/道路等；采纳市/区县/省/兴趣点；全拒则 place/text 景点/行政区 POI → 父级 `cityname` + 坐标。禁止鼓浪屿→厦门源码表（ADR-042）。

**Do not:** where2play 客户端改城市名；把酒店/餐饮 POI 当目的地。
