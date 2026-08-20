---
title: Google Places Photos media URL 两种模式
type: ops-lesson
status: active
as_of: 2026-08-20
tags:
  - google-maps
  - photos
  - api
related:
  - knowledge/maps/vendor-adapters.md
  - adr/ADR-026-region-based-provider-auto-selection.md
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

### 两种 URL 模式

| 模式 | URL 参数 | 返回 | 用途 |
|------|---------|------|------|
| **重定向（前端用）** | `?maxWidthPx=400&key={KEY}` | 302 → 图片 CDN | `<img src>` 直接使用 |
| **元数据（后端用）** | `?maxWidthPx=400&skipHttpRedirect=true` | JSON `{ photoUri, ... }` | 服务端获取真实 URL |

### 注意事项

- **API key 暴露**：重定向模式的 URL 包含明文 API key。MVP 阶段可接受；生产应通过 BFF 代理
- **费用控制**：每次 `<img>` 加载 = 一次 Photo Place Details 计费。限制返回 3 张（`photos.slice(0, 3)`）
- **Feature flag**：`GOOGLE_PHOTOS_ENABLED=false` 关闭 Google 图片（AMAP 图片不受影响）
- **AMAP 图片**：`show_fields=photos` 直接返回 `photos[].url`，无需拼接，无 key 问题

## Links

- [vendor-adapters](./vendor-adapters.md) — 各 vendor 字段覆盖
- Google Places Photos 文档: https://developers.google.com/maps/documentation/places/web-service/place-photos
