# MVP-T1 收尾台账

**Branch:** `real-agent-refactory`  
**Scope:** `agent-itinerary-93a` / `2play-plan-90a` + quality `95`–`99`（含 `96` / ADR-060）  
**Not in scope:** MVP-T2 全环、what2eat、未请求的 git commit

每确认一项，在下表写 **Confirmed / 日期 / 说明**。

| # | 项 | 状态 | 日期 | 说明 |
| --- | --- | --- | --- | --- |
| 1 | 功能确认可用 | **Confirmed** | 2026-09-09 | 用户确认 Lisbon 8 项 + 4 问 + 芯片 usable（芯片可少/空，ADR-060）；不再 Playwright 重验 |
| 2a | 合并设计进 `agent-design.md` | **Confirmed** | 2026-09-09 | 真智能体能力/原则 + MVP-T1 as-built 并入；去重 T1 节 |
| 2b | 对照实现改链接 / 纠偏 | **Confirmed** | 2026-09-09 | backlog / architecture / ADR-050·055·056·060 / stories / README / change-log 改锚 `agent-design`；芯片 = collected 可空 |
| 2c | 冗余文件删/ stub | **Confirmed** | 2026-09-09 | 删 `tmp-0909.md`；`real-agent-refactory.md` / `draft-nominate-must-see-prompt.md` → 短指针 |
| 3 | 质量门（tsc / 测试 / retrospective） | **Confirmed** | 2026-09-09 | places-agent / where2play `tsc --noEmit` 绿；`plan-trip.test.ts` 23 pass；where2play plan-constraints / plan-intake / plan-page 46 pass；`make check-mvp10-css` OK；retro：ADR-059/060 已有，补 knowledge `intake-no-forced-nominate` |

## 变更仓列表（等你下令再 commit）

| 仓 | 预期变更 |
| --- | --- |
| umbrella `places-workspace` | `mvp-1t-closing-plan.md`、`agent-design.md`、`plan.md`、`product-backlog.md`、`2.architecture.md`、`change-log.md`、链接修正、删 `tmp-0909.md`、stub `real-agent-refactory` / `draft-nominate`、knowledge `intake-no-forced-nominate` |
| `places-agent` | tsc 类型修复（`plan-trip` budget、`DayStopLike`、`dispatch`、相关测试/脚本） |
| `where2play` | tsc（`ChatLlmEnv`、`SlotPreviewPayload.reason?`、`tsconfig` 排除 vitest.config）；约束条等既有工作树 |

## 下一步（本收尾结束后问你）

- ~~是否三仓 commit / push~~ **Done 2026-09-09**（`real-agent-refactory`）
- 是否开 MVP-T2

| 仓 | Commit |
| --- | --- |
| umbrella | `9e7ffca` docs: close MVP-T1 — merge true-agent design, usable confirmed |
| places-agent | `5fd0bd7` feat: harden plan_trip intake and clear TypeScript gates |
| where2play | `bb861bf` feat: ship MVP-T1 plan_trip intake UI and constraints panel |
