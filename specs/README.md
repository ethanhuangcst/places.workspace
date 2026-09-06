# places-workspace specs

Umbrella specs for the **places** product family. Parent git tracks this tree only; product code lives in sibling repos (`places-agent/`, `what2eat/`, `where2play/`), each with its own remote.

## Layout

| Path | Role |
| --- | --- |
| [`1.req-specs.md`](./1.req-specs.md) | Agent / family requirements entry |
| [`2.architecture.md`](./2.architecture.md) | Trust boundaries, deploy shape, workspace remotes, ADR index |
| [`3.tech-specs.md`](./3.tech-specs.md) | Locked stack versions |
| [`6.deployment-plan.md`](./6.deployment-plan.md) | Family deploy overview |
| [`adr/`](./adr/) | Binding decisions |
| [`knowledge/`](./knowledge/) | Reusable lessons (not binding AC) |
| [`agent-specs/`](./agent-specs/) | places-agent product specs (stories, design, test, e2e) |
| [`2eat-specs/`](./2eat-specs/) | what2eat product specs |
| [`2play-specs/`](./2play-specs/) | where2play product specs |

**`specs/` = family specs + three product specs.** ADR and knowledge stay here.

## Non-contract (do not treat as AC)

| Path | Notes |
| --- | --- |
| [`samectx-notes/`](./samectx-notes/) | Session dumps; historical paths may be stale |
| `*-chat.md` / chat exports | Conversation archives, not product contract |
| `agent-specs/e2e-test-results/*.{png,json}` | Probe binaries; `.md` result write-ups may remain |

## Product code (sibling dirs, ignored by parent git)

| Directory | Specs |
| --- | --- |
| `../places-agent/` | [`agent-specs/`](./agent-specs/) |
| `../what2eat/` | [`2eat-specs/`](./2eat-specs/) |
| `../where2play/` | [`2play-specs/`](./2play-specs/) |

True-agent refactor entry: [`agent-specs/real-agent-refactory.md`](./agent-specs/real-agent-refactory.md).
