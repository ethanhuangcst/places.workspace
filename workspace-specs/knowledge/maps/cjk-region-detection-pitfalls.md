---
title: CJK 文本不等于中国大陆 — 区域检测陷阱
type: ops-lesson
status: active
as_of: 2026-08-20
tags:
  - provider
  - region-detection
  - cjk
  - google-geocode
related:
  - adr/ADR-026-region-based-provider-auto-selection.md
  - adr/ADR-030-geocode-first-region-detection.md
  - knowledge/ops/mvp3a-provider-auto-selection.md
---

# CJK 文本不等于中国大陆 — 区域检测陷阱

## Summary

用"CJK 字符占比 >30%"判断地址是否在中国大陆，误判率极高。日文汉字（銀座）、韩文汉字词（明洞）、港台繁体（中環、臺北）全部 100% CJK 但不在大陆。此规则已在 ADR-030 中删除，改为 Google Geocode 先行判断。

## Evidence

| 输入 | CJK 占比 | 实际位置 | CJK 规则判定 | 正确? |
|------|---------|---------|-------------|------|
| "中環" | 100% | 香港 | mainland ❌ | ❌ |
| "銀座" | 100% | 东京 | mainland ❌ | ❌ |
| "明洞" | 100% | 首尔 | mainland ❌ | ❌ |
| "臺北遠山" | 100% | 台湾 | mainland ❌ | ❌ |
| "新加坡滨海湾" | 83% | 新加坡 | mainland ❌ | ❌ |
| "北京银河SOHO" | 50% | 北京 | mainland ✅ | ✅ |

5/6 误判。规则的唯一成功案例（北京）用城市列表就能判断，不需要 CJK 占比。

## Lesson / guidance

1. **永远不要用字符编码范围做地理判断** — CJK 统一汉字被中日韩越共用
2. **Google Geocode 是最可靠的地名消歧** — 格式化地址包含国家，准确率接近 100%
3. **Marker 列表需要繁简体都收录** — "台北" ≠ "臺北"，都要有
4. **中国 bounding box 太大** — lat 18-54, lng 73-135 覆盖了韩国、蒙古北部、缅甸北部。坐标判断不能作为唯一手段
5. **默认 fallback 应该是 Google（"other"）** — Google 自带消歧能力，AMAP 只懂中国大陆

## Links

- [ADR-030](../../adr/ADR-030-geocode-first-region-detection.md) — 正式决策
- [ADR-026](../../adr/ADR-026-region-based-provider-auto-selection.md) — 原始方案（CJK 规则已被取代）
