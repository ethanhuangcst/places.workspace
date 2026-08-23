---
title: Cross-product spec drift (agent vs 2play)
type: ops-lesson
status: active
as_of: 2026-08-23
tags:
  - where2play
  - places-agent
  - documentation
  - Mode-H
  - ADR-037
  - ADR-039
related_spec: workspace-specs/2.architecture.md
related:
  - adr/ADR-037-where2play-plan-l2-quanzil.md
  - adr/ADR-039-cross-product-as-built-vs-target.md
  - ../../1.places-agent/agent-specs/performance.md
  - ../../3.where2play/2play-specs/itinerary-design.md
---

# Cross-product spec drift：为什么两边文档不同步、决策像不一致

## Summary

agent 与 2play 规格曾对 Mode H / L2 管线给出互相矛盾的「已实现 / 未实现」。根因不是产品拍板冲突（ADR-037 / Mode H 方向一致），而是**状态混写**：把「agent 能力交付」「2play 是否消费」「文档里的目标架构」三件事写成同一个布尔值，且关闭 MVP-8 时未回写 consumer 与 `performance.md`。

## Evidence（2026-08-23 对齐时）

| 表象 | 实际 |
| --- | --- |
| `agent-stories` Feature **35** = Done | HTTP/MCP `execution=host` 已可测 |
| `performance.md` / itinerary §1 仍写 Mode H 未实现或目标图当现状 | 共享状态表与专项设计未随 Wave C 关闭更新 |
| 2play stories「依赖 agent 35」仍 To-do 阻塞 | agent 已 Done；缺口是 **2play `plan-11`** |
| itinerary-design 序列图：BFF→agent host | **As-built** 是本地 `buildArrangeDayMessages` + OPENAI_CN |
| ADR-037「一套 prompt」 | 意图正确；实现上曾短期双份拼装（agent + 2play） |
| 用户三步核对（discover → host prompt → 2play LLM） | 与 Mode H **一致**；与 MVP-2 **as-built**（跳过 host HTTP）不一致 |
| MVP-2 E2E 绿 | 只验保存闭环；**不**验 host / 地标 / 真交通 → MVP-2 Done ≠ Mode H Done |
| 2026-08-23 文档波 | **MVP-3** 收 31–33；Chat→**MVP-4**；Replan→**MVP-5**；ADR-037 增 as-built 表 |

## 为什么会漂（过程）

1. **三仓独立进度：** `1.places-agent` / `3.where2play` / `workspace-specs` 可各自改 stories；交叉引用靠人手。  
2. **关闭 Wave 只勾本仓：** MVP-8 DoD 更新了 agent-stories，未强制刷 `performance.md` §3.1/§11 与 2play 阻塞列。  
3. **目标态文档先写：** Progressive / Mode H 设计以目标管线叙述，缺少 as-built 节，读者当成生产真相。  
4. **「依赖」语义过期：** Feature 号依赖在 producer Done 后应变「消费故事」，否则计划表永远「并行阻塞」。  
5. **同名不同义：** 「Mode H 落地」可指 handoff 契约、MCP 默认 host、或 2play 已换 prompt 源——未限定主语就会冲突。  
6. **ADR 措辞歧义：** 「不再调用 `arrange_day`」若未写清 **execution=agent**，会被实现成「零 HTTP」，与 host 取 prompt 冲突；契约测 `never_call_agent_arrange_day` 加剧分叉直至 **MVP-3** 改测 `must_call_host`。

## Lesson / guidance

1. 跨产品能力一律按 [ADR-039](../../adr/ADR-039-cross-product-as-built-vs-target.md)：**Producer / Consumer / As-built / Target**。  
2. 关闭 producer Feature 的文档清单（最低）：  
   - 本仓 stories + test-plan  
   - `performance.md`（若该能力在性能方案里有状态行）  
   - consumer stories 依赖/阻塞列  
   - 相关 ADR Consequences 或 as-built 表  
3. 专项设计（如 `itinerary-design.md`）开头固定两框：as-built / target。  
4. 排障先问：「你说的 Done 是 agent、2play，还是文档目标？」再查代码路径。

## Links

- [ADR-039](../../adr/ADR-039-cross-product-as-built-vs-target.md)  
- [ADR-037](../../adr/ADR-037-where2play-plan-l2-quanzil.md)  
- [performance.md](../../../1.places-agent/agent-specs/performance.md)  
- [itinerary-design.md](../../../3.where2play/2play-specs/itinerary-design.md)

## Sync note (2026-08-23)

As-built dual-channel contract consolidated in [ADR-043](../../adr/ADR-043-chatbox-mcp-and-cross-product-closure.md): 2play = Mode H + enrich; ChatBox MCP = force agent. Prefer that ADR over older “target Mode H / plan-13 未做” wording.
