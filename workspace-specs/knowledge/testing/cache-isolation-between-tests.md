---
title: 测试间缓存隔离 — 模块级单例缓存的陷阱
type: ops-lesson
status: active
as_of: 2026-08-21
tags:
  - testing
  - cache
  - vitest
related:
  - knowledge/testing/vendor-live-vs-fixture.md
---

# 测试间缓存隔离 — 模块级单例缓存的陷阱

## Summary

places-agent 添加 geocode-cache 和 search-cache（模块级 Map 单例）后，3 个原本通过的 HTTP 集成测试开始失败。原因：测试 A 的搜索结果被缓存 → 测试 B 读到了 A 的缓存结果而非自己 fixture 的数据。

## Evidence

- `TC-H05` (TripAdvisor enrich): 缓存返回了无 enrich 的结果（来自前序测试）
- `TC-H15` (Worker fallback): 缓存返回了 direct 路径的结果
- `should_not_fallback_to_google_when_caller_forces_amap_empty`: 缓存返回了前序的 Google 结果

## Lesson / guidance

1. **模块级单例缓存 + vitest = 跨测试污染** — vitest 在同一进程中运行所有测试，模块只加载一次，Map 实例在测试间共享
2. **必须在 `beforeEach` 或 `afterEach` 中清理缓存：**
```typescript
afterEach(() => {
  clearSearchCache();
  clearGeocodeCache();
});
```
3. **导出 `clear*()` 函数** — 缓存模块必须暴露清理接口，否则测试无法隔离
4. **不要用 `vi.resetModules()`** — 会导致类型不匹配和其他副作用；显式 clear 更可靠

## Links

- [vendor-live-vs-fixture](./vendor-live-vs-fixture.md) — fixture 相关测试隔离
