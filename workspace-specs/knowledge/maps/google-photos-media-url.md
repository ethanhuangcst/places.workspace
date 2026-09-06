---
title: Google Places Photos media URL 两种模式
type: ops-lesson
status: active
as_of: 2026-09-06
tags:
  - google-maps
  - photos
  - api
related:
  - knowledge/maps/vendor-adapters.md
  - adr/ADR-051-discover-resolve-display-photo.md
  - adr/ADR-053-origin-stay-as-stop-card.md
---

# Google Places Photos media URL 两种模式

## Summary

Google Places API (New) 的 photos 字段返回 `photos[].name`（如 `places/{id}/photos/{ref}`），需拼接为 media URL 才能获取图片。有两种模式，选错会导致前端图片无法显示。

## Evidence

1. **初版错误**：使用 `skipHttpRedirect=true` 生成 URL，浏览器 `<img>` 显示为破碎图标
   - 原因：该参数使 API 返回 JSON 元数据（含 `photoUri`），而非图片二进制
2. **修复**：去掉 `skipHttpRedirect`，加上 `key={API_KEY}`
   - URL 直接 302 重定向到 CDN 图片地址，`<img>` 可用

## Lesson / guidance

### ADR-051 / ADR-053（2026-09-06）

places-agent **不得**把带 `key` 的 media URL 写入 Trip `photos[]`。正确路径：搜索卡保留 `google_photo_names` → 服务端 `skipHttpRedirect=true` 取 **`photoUri`（CDN）** → 写入可展示 `photos[0]`。

| 卡类型 | 何时解析 |
|--------|----------|
| 景点 | `discover_places` |
| 正餐 | `plan_next_stop` 填站 |
| 起点 stay | **intake 建卡**时解析（ADR-053）；fill 有指针则只抄，无指针才现搜（禁 `cards[0]`） |

列表拇指与 place-sheet lightbox **共用**同一 `photos[0]`。服务端 Google media 请求宽度 **`maxWidthPx=800`**（原 400 ×2）。见 [ADR-051](../../adr/ADR-051-discover-resolve-display-photo.md)、[ADR-053](../../adr/ADR-053-origin-stay-as-stop-card.md)。

### 两种 URL 模式

| 模式 | URL 参数 | 返回 | 用途 |
|------|---------|------|------|
| **重定向（前端用）** | `?maxWidthPx=800&key={KEY}` | 302 → 图片 CDN | `<img src>` 直接使用（产品路径勿把带 key 的 URL 写入账本） |
| **元数据（后端用）** | `?maxWidthPx=800&skipHttpRedirect=true` | JSON `{ photoUri, ... }` | 服务端获取真实 URL → 写入 `photos[0]` |

### 注意事项

- **API key 暴露**：重定向模式的 URL 包含明文 API key。账本只存无 key 的 `photoUri` / 高德直链（ADR-051 D4）。
- **费用控制**：每次解析 = 一次 Photo 计费。每卡最多一张可展示图；同 trip 复用已解析卡。
- **Feature flag**：`GOOGLE_PHOTOS_ENABLED=false` 关闭 Google 图片（AMAP 图片不受影响）
- **AMAP 图片**：`show_fields=photos` 直接返回 `photos[].url`，无需拼接，无 key。搜索常给 **`http://store.is.autonavi.com/showpic|query_pic`**。账本只收 https：对 `*.autonavi.com` / `*.amap.com` **升 https**（[ADR-051](../../adr/ADR-051-discover-resolve-display-photo.md) D6）。其它 http 仍丢弃。实现：`upgradeAmapInsecurePhotoUrl`（`amapPoiToCard` + `resolveDisplayPhoto`）。

## Links

- [vendor-adapters](./vendor-adapters.md) — 各 vendor 字段覆盖
- Google Places Photos 文档: https://developers.google.com/maps/documentation/places/web-service/place-photos
