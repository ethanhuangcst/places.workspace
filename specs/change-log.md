# Specs change log

从 **2026-09-07** 起记录本仓库 `specs/`（及紧密绑定的 agent 测试/策略）修订。条目按日期倒序；每条写清 **改了什么** 与 **为什么**。

格式：

```text
## YYYY-MM-DD — 短标题

- **范围：** …
- **变更：** …
- **关联：** ADR / 文件路径
```

---

## 2026-09-07 — POC 签收（agent-poc-01 Done）

- **范围：** 真智能体 POC 标 Done；工作计划切到 MVP-T1。
- **变更：**
  - [`product-backlog.md`](./product-backlog.md) §0 / §1：`agent-poc-01` → Done；下一步 MVP-T1。
  - [`plan.md`](./plan.md)：POC 待办全部勾选；下一步改为写 MVP-T1 AC。
  - [`agent-stories.md`](./agent-specs/agent-stories.md)：`agent-poc-01` 状态 Done；范围注明全环超出原 ADR-054 D2。
  - [`real-agent-refactory.md`](./agent-specs/real-agent-refactory.md)：POC 签收行改为已完成。
  - 验证脚本补质量 mock（时钟链 / 下午站 / 餐店卡）与检查项；产物 [`poc-true-agent-verification.html`](./poc-true-agent-verification.html)。
  - 知识：[`knowledge/agent/poc-verify-fixture-quality.md`](./knowledge/agent/poc-verify-fixture-quality.md)。
- **关联：** ADR-054、ADR-056、ADR-057、ADR-058；`places-agent/scripts/verify-poc-true-agent.ts`。

## 2026-09-07 — POC 全环模型驱动 + 无 Google 验证

- **范围：** `plan_trip` 全环改为模型 act-or-stop；fixture 验证不调 Google。
- **变更：**
  - `runFullLoopAgent` 为默认全环：模型选 `resolve_origin_stay` / `search_places` / `make_itinerary` / `plan_next_stop` / `commit_artifacts` / `stop`；全环发现的 eligible POI 回填 registry。
  - 旧 `runFullLoop` 固定管线保留为 `PLAN_TRIP_LEGACY_FULL_LOOP=1` 回退。
  - 新增 `places-agent/scripts/verify-poc-true-agent.ts`（进程内、fixture、Prisma Lisbon 池）；结果写入 [poc-true-agent-verification.md](./poc-true-agent-verification.md)。
  - [real-agent-refactory.md](./agent-specs/real-agent-refactory.md) G8 行标记已实现。
- **关联：** ADR-054、ADR-056、ADR-057、ADR-058；`plan-trip.ts`。

## 2026-09-07 — 跨供应商同地点去重交由 LLM（ADR-058）+ 真智能体改进路线

- **范围：** stops pool 跨供应商去重策略；POC 真智能体合规路线。
- **变更：**
  - 新增 [ADR-058](./adr/ADR-058-cross-provider-duplicate-llm-judges.md)：跨供应商同一地点（香港 Google+AMAP）不写库合并、不读池硬去重，去重由环内 LLM 取点时自行判断；prompt 硬约束 + 事实闸兜底。
  - 更新 [agent-design.md](./agent-specs/agent-design.md) §9.2 / §5.2：明确 LLM 取点需自己判断跨供应商同地点。
  - [real-agent-refactory.md](./agent-specs/real-agent-refactory.md) 追加「下一步改进」节：列出 G6/G7/G8/F89/G5 + POC 签收待决项、建议顺序、结合 ADR-057 三层测试的验证矩阵。
- **关联：** ADR-049、ADR-050、ADR-052、ADR-056、ADR-057、ADR-058；`destination-poi-registry.ts`；`plan-trip.ts`。

## 2026-09-07 — 成本节约测试策略入库 + change-log 启动

- **范围：** agent 测试策略文档化；修订记录机制启动。
- **变更：**
  - 新增 [ADR-057](./adr/ADR-057-cost-conscious-agent-test-strategy.md)：L0 fixture CI、L1 probe 文件缓存、L2 日配额门、stops pool 作 live feed（非城市百科）。
  - 更新 [agent-test-plan.md](./agent-specs/agent-test-plan.md)：绑定表、§1.2、§9 CI 门控、§11 清单。
  - 新建本文件，作为此后 specs 修订的单一入口；[`README.md`](./README.md) Layout 表增加 `change-log.md` 行。
- **关联：** ADR-021、ADR-042、ADR-056、ADR-057；`.github/workflows/places-agent-tests.yml`；`places-agent/scripts/seed-city-pois.ts`、`probe-budget.ts`、`search-cache.ts`。

## 2026-09-07 — Registry 回填语义 + 三城 stops pool 种子

- **范围：** AttractionPoi 写路径与测试 feed。
- **变更：**
  - 新增 [ADR-056](./adr/ADR-056-registry-backfill-semantics.md)：唯一性 `(provider, native_id)`；匹配无差跳过 / 有差更新+aliases / 无匹配新增；`cardSlim` 存 photos、不存 `must_see`；`plan_trip` commit 接线 `safeUpsertEligiblePois`。
  - 更新 [real-agent-refactory.md](./agent-specs/real-agent-refactory.md) 挂钩表与落库节。
  - 种子脚本建 Lisbon / Hong Kong / Taipei 池各 ≥100（有图优先）；删除旧「里斯本」「香港」碎片 destination。
- **关联：** ADR-049、ADR-051、ADR-056；`destination-poi-registry.ts`、`plan-trip.ts`。
