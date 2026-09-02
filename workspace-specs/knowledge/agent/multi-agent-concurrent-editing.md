---
title: 多 Agent 并发编辑同一仓库的协调
type: ops-lesson
status: active
as_of: 2026-09-01
tags:
  - agent
  - multi-agent
  - coordination
  - git
related:
  - knowledge/agent/places-agent-loop.md
---

# 多 Agent 并发编辑同一仓库的协调

## Summary

places-workspace 常有多 个 agent 同时开发 places-agent / where2play，共享 `create-server.ts`、`dispatch.ts`、`schemas.ts` 等"热文件"。并发直接改同一文件会互相覆盖、丢失工作。本笔记记录一次有效的协调策略。

## Evidence

- MVP-12（ADR-045）实现 F49–F52 时，另外两个 agent 正在活跃编辑 F49/F50 共享文件。
- 直接重写热文件会破坏他人未提交的改动；完全等待则阻塞交付。
- 用户先选 "nonconflict"（只做不冲突的工作），后明确指示 "完成所有 to-do"，等同授权接管热文件。

## Lesson / guidance

1. **先审计再动手** — 用 `git status` + 文件 mtime 比对计划，区分"已完成 / 缺失 / 被他人改动"，避免重复或覆盖。
2. **优先不冲突的工作** — 先做自包含新模块（新文件 + 独立测试）、修复陈旧非热文件的类型错误，把热文件留到最后。
3. **接管热文件需显式授权** — 不要擅自重写他人正在编辑的文件；只有用户明确指示"完成全部"时才接管，并一次性集中改完。
4. **接管后集中收尾** — 接管后一次性完成所有热文件改动 + 接线 + 测试，避免多次穿插造成新的并发窗口。
5. **以 tsc + 全量测试为收口** — 并发改动后必须 `tsc --noEmit` 干净 + 全量 `vitest` 绿，确认未破坏他人代码。
6. **偶发失败先排查是否为既有 flake** — 全量套件偶现 1 例失败但重跑稳定绿时，多为 DB-setup 竞态等既有 flake，非新代码引入；定位到具体测试再决定是否修。

## Links

- [places-agent-loop](./places-agent-loop.md) — agent 主循环模式
- [cache-isolation-between-tests](../testing/cache-isolation-between-tests.md) — 并发开发中引入新缓存时的测试隔离
