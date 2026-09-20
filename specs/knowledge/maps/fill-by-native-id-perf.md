---
title: Fill by native_id — 墙钟预期
type: ops-lesson
status: active
as_of: 2026-09-20
tags:
  - fill
  - plan_next_stop
  - performance
related:
  - adr/ADR-072-stop-identity-provider-native-id.md
  - adr/ADR-051-discover-resolve-display-photo.md
  - knowledge/maps/google-restaurant-search-latency.md
---

# Fill by native_id — 墙钟预期

**ADR-072 D5。** 按指针抄池卡纠正译名缺图；不是填站主加速手段。

## Hop 组成（串行）

| 步骤 | 按 id 能否省 |
| --- | --- |
| `resolvePoint` 名 miss → geocode | 能（池卡有坐标） |
| Directions × 3（walk / transit / drive） | 不能 |
| 池 miss → `searchPlaces` | 能 |
| 无图 → Details / media | 池已有 https 则能 |
| 正餐环搜 | 不能（ADR-049）。Google 墙钟见 [`google-restaurant-search-latency.md`](./google-restaurant-search-latency.md) |

## Live spike（2026-09-19，locale CN，3 天）

| 目的地 | 墙钟 | 站数 | stay | 景点/餐 |
| --- | --- | --- | --- | --- |
| 台北 | ~60.8s | 20 | 12–16ms | 0.4–2.1s |
| 里斯本 | ~48.9s | 22 | 12–27ms | 景点 0.4–1.4s；lunch 最高 ~2.7s |

## Envelope fill 刷新（2026-09-20）

`plan_trip` 全环 `timing.fill_s`（现行 ADR-049 餐搜 + Directions，非 A/B/C）：杭州 **15.43s / 15 站**；台北 **67.81s / 18 站**。详见 [`../agent/skeleton-meals-vs-fill-eval.md`](../agent/skeleton-meals-vs-fill-eval.md)。

stay 已是池命中 + `origin_mode`。景点慢两个数量级，主因算路，不是抄 `photos[0]`。

## 粗算

- 乐观（几乎每站译名 miss）：12–16 景点 × 省 search±geocode±补图 ~0.4–1.0s → 整趟 **约 5–16s（~10–25%）**。
- 仅 Belém / Jerónimos 两站：**约 1–2s（&lt;5%）**。
- 高德名与池一致：增量 **~0**。
- Google fill 另打 Details 取 UI 显示名：若不能与补图 Details 合并，约 **+0.2–0.4s × 站数**，会吃掉部分节省。

## 验收

写「有指针则图从池抄」。不要写「填站显著变快」。Directions / 餐搜加速另开故事。
