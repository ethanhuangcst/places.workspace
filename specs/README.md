# places-workspace specs

Umbrella specs for the **places** product family. Parent git tracks this tree only; product code lives in sibling repos (`places-agent/`, `what2eat/`, `where2play/`), each with its own remote.

## Read first

| Path | Role |
| --- | --- |
| [`product-backlog.md`](./product-backlog.md) | **唯一**产品级概要需求与排期真源（§0 当前下一步 · §1 功能表 · §3 原则 · §5 产品定义） |
| [`plan.md`](./plan.md) | 工作计划与下一步（真智能体重构插入） |
| [`agent-specs/real-agent-refactory.md`](./agent-specs/real-agent-refactory.md) | True-agent Target 设计（`plan_trip` + fetch；含能力清单） |

## Layout

| Path | Role |
| --- | --- |
| [`product-backlog.md`](./product-backlog.md) | **唯一**产品级概要需求与排期真源 |
| [`2.architecture.md`](./2.architecture.md) | Trust boundaries, deploy shape, workspace remotes, ADR index |
| [`3.tech-specs.md`](./3.tech-specs.md) | Locked stack versions |
| [`6.deployment-plan.md`](./6.deployment-plan.md) | Family deploy overview |
| [`adr/`](./adr/) | Binding decisions |
| [`knowledge/`](./knowledge/) | Reusable lessons (not binding AC); historical refactor log: [`knowledge/agent/refactor-plan-archive.md`](./knowledge/agent/refactor-plan-archive.md) |
| [`agent-specs/`](./agent-specs/) | places-agent GWT/AC、设计、测试、e2e |
| [`2eat-specs/`](./2eat-specs/) | what2eat GWT/AC、设计、测试 |
| [`2play-specs/`](./2play-specs/) | where2play GWT/AC、设计、测试 |

**`specs/` = family specs + three product specs.** ADR and knowledge stay here.  
**排期只写在 `product-backlog.md`；三个 `*-stories.md` 只写 GWT/AC。**

## Non-contract (do not treat as AC)

| Path | Notes |
| --- | --- |
| [`samectx-notes/`](./samectx-notes/) | Session dumps; historical paths may be stale |
| `*-chat.md` / chat exports | Conversation archives, not product contract |
| `agent-specs/e2e-test-results/*.{png,json}` | Probe binaries; `.md` result write-ups may remain |
| `_scratch-*.md` | Temporary generation scratch; delete after merge |

## Product code (sibling dirs, ignored by parent git)

| Directory | Specs |
| --- | --- |
| `../places-agent/` | [`agent-specs/`](./agent-specs/) |
| `../what2eat/` | [`2eat-specs/`](./2eat-specs/) |
| `../where2play/` | [`2play-specs/`](./2play-specs/) |
