---
title: MVP-3a Provider 自动选择实现经验
type: ops-lesson
status: active
as_of: 2026-08-20
tags:
  - provider
  - search
  - region-detection
  - what2eat
related:
  - adr/ADR-026-region-based-provider-auto-selection.md
  - adr/ADR-005-caller-driven-providers.md
  - knowledge/ops/safari-secure-cookie-localhost.md
---

# MVP-3a Provider 自动选择实现经验

## Summary

places-agent 从 caller-driven provider 选择切换为 agent 端自动选择。核心经验：
caller 不应维护 provider 路由逻辑，这导致了 what2eat 的北京搜到美国餐厅 bug。

## Evidence

1. **根因定位**：what2eat `providersForPin()` 对大陆传 `["AMAP", "GOOGLE_MAPS"]`，Google 对中国地址返回美国结果
2. **修复路径**：places-agent 新增 `provider-resolver.ts`（三区域检测：大陆/香港/其他），what2eat 不再传 `providers`
3. **台湾陷阱**：初版 CJK 字符比 >30% 的启发式将台湾地址（`台北市信義區`）误判为大陆 → 需要显式台湾排除列表
4. **测试验证**：live smoke test 确认大陆→AMAP only (20/20 in China)，香港→Google+AMAP (20+20)

## Lesson / guidance

### 区域检测优先级

```
1. 台湾标记排除 → "other"（最先检查，避免 CJK 误判）
2. 香港标记 → "hongkong"
3. 中国城市列表 → "mainland"
4. CJK 字符比 >30% → "mainland"（兜底，台湾已排除）
5. 坐标范围：台湾排除 → 香港框 → 中国框
6. 以上均不匹配 → "other"
```

### Caller 解耦原则

- Caller（what2eat, where2play）**不应**维护 provider 路由逻辑
- Caller 传 `address` 文本即可，agent 端完成检测
- 显式 `providers[]` 仍然生效，用于调试和特殊场景

### 可 ingest 到 KB 的知识点

| 知识点 | 适合 KB 位置 | 说明 |
|--------|-------------|------|
| 三区域 provider 策略（大陆/香港/其他） | agent/places-agent-loop.md | 更新 provider 选择逻辑 |
| 台湾排除列表（AMAP 覆盖差） | maps/vendor-adapters.md | AMAP 不覆盖台湾 |
| what2eat 不再传 providers | web-app-development/ | caller 解耦模式 |
| SessionManager TTL 清理 | ops/places-agent-next-runtime.md | MCP/SSE session 生命周期管理 |
| readJsonBody 安全化 | ops/places-agent-next-runtime.md | 防止非 JSON body crash |
| graceful shutdown (SIGTERM/SIGINT) | ops/places-agent-local-daemon.md | 进程信号处理 |
| Specs 整合经验（9→4 文件） | web-app-development/ | 文档管理最佳实践 |

## Links

- [ADR-026](../adr/ADR-026-region-based-provider-auto-selection.md) — 正式决策
- [ADR-005](../adr/ADR-005-caller-driven-providers.md) — 被部分取代
- [Safari cookie 经验](./safari-secure-cookie-localhost.md) — 同一 session 的另一个修复
