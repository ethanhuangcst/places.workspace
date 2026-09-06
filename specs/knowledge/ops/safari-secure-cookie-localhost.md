---
title: Safari 拒绝 HTTP localhost 上的 Secure cookie
type: ops-lesson
status: active
as_of: 2026-08-20
tags:
  - safari
  - cookie
  - auth
  - dev-environment
related:
  - adr/ADR-025-places-agent-postgres-prisma.md
  - knowledge/ops/places-agent-local-daemon.md
  - knowledge/web-app-development/lessons-from-places-agent-mvp1.md
---

# Safari 拒绝 HTTP localhost 上的 Secure cookie

## Summary

places-agent 从 SQLite 迁移到 Postgres 后，Safari 无法登录 `http://localhost:3010`。
根因：`@next/env` 的 `loadEnvConfig()` 在 dev 启动时加载了 `.env.production`（其中 `NODE_ENV=production`），
导致 session cookie 被标记为 `Secure`。Chrome 对 localhost 豁免该限制，Safari 严格遵守规范，
拒绝在非 HTTPS 连接上存储 `Secure` cookie，登录 API 返回 200 但 cookie 被丢弃。

## Evidence

1. `curl -D -` 抓取 login 响应 → `Set-Cookie: ...; Secure; HttpOnly; SameSite=lax`
2. `writeSession()` 中 `secure = process.env.NODE_ENV === "production"` → dev 下应为 `false`
3. `node -e "loadEnvConfig(cwd()); console.log(process.env.NODE_ENV)"` → 输出 `production`
   - `loadEnvConfig` 同时加载了 `.env.local` 和 `.env.production`
   - `.env.production` 含 `NODE_ENV=production`，覆盖了默认值
4. Chrome 对 `localhost` 有 [Secure cookie 豁免](https://bugs.chromium.org/p/chromium/issues/detail?id=823903)，Safari 没有

## Lesson / guidance

### 根因修复

在 `scripts/dev-server.sh` 中于 `exec` 前显式 `export NODE_ENV=development`。
这保证 `loadEnvConfig` 按 development 模式加载，不会读取 `.env.production`。

### 排查清单（本地 HTTP + cookie 场景）

1. **抓 Set-Cookie header** — 检查 `Secure` / `SameSite` / `Domain` / `Path`
2. **确认 NODE_ENV** — `loadEnvConfig` / dotenv 可能加载了非预期的 env 文件
3. **跨浏览器测试** — Chrome 对 localhost 的 Secure cookie 静默豁免，不代表其他浏览器也如此
4. **不要仅凭 API 返回 200 就认为登录成功** — 还需确认 cookie 是否被浏览器实际存储

### 额外发现

- 迁移到 Postgres 后 `prisma db seed` 用 `DEV_ADMIN_PASSWORD=devpass` 重新 hash 了密码，
  旧 SQLite 中的密码（如 `12345678`）不会被自动迁移。调试时应先验证密码是否匹配，
  而非假设密码与迁移前一致。

## Links

- [ADR-025 places-agent-postgres-prisma](../adr/ADR-025-places-agent-postgres-prisma.md)
- [places-agent-local-daemon](./places-agent-local-daemon.md)
- Chromium Secure cookie localhost 豁免: https://bugs.chromium.org/p/chromium/issues/detail?id=823903
