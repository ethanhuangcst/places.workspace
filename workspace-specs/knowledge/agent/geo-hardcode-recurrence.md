---
title: 地理知识硬编码重犯 — 餐厅路由、discover CATALOG 与五处别的门
type: ops-lesson
status: active
as_of: 2026-08-23
tags:
  - places-agent
  - discover_places
  - what2eat
  - anti-pattern
  - retrospective
related:
  - ../../adr/ADR-042-no-city-encyclopedia-in-source.md
  - ../../adr/ADR-038-discover-places-quality.md
  - ../../adr/ADR-026-region-based-provider-auto-selection.md
  - ../../adr/ADR-030-geocode-first-region-detection.md
  - ./discover-lisbon-ab-probe.md
  - ./discover-places-quality-seed-filter.md
  - ../ops/mvp3a-provider-auto-selection.md
---

# 地理知识硬编码重犯 — 为什么犯过的错又来一次

## Summary

同一反模式出现三次：把**目的地理知识**写进源码/调用方表来「提高质量」。第一次是餐厅侧硬编码 provider/区域逻辑（北京→美国餐厅）；第二次是 `discover-must-see` 按城 `CATALOG`（里斯本无 must-see）；第三次是 CATALOG 之外的四处「别的门」——must-see 簇名单、地标去重正则、地标名后缀正则、远郊行政区过滤。第三次不是「忘了合入里斯本种子」，而是**ADR-042 只禁了文件名（`discover-must-see.ts`）没禁原则（源码任何城市 POI 知识），新硬编码在 dedup/coverage/name-split/district-filter 等不同名目下被加进别的文件**，逃出原 ADR 视野，且无全局守卫测试。

## Evidence

### 第一次（餐厅）

- what2eat `providersForPin()` 等对大陆硬传双源 → Google 对中文地址返回美国餐厅（ADR-026 / `mvp3a-provider-auto-selection`）。
- 后续又用城市列表、CJK 启发式、海外 exclusion 表修补——仍是「表驱动地理」，再由 ADR-030 推向 geocode-first。
- **当时学到的：** 地理路由不应散落在 caller；启发式/表会漏未见过的目的地。

### 第二次（discover must-see）

- ADR-038 Accepted「国内热门城白名单 seed」；实现为 `CATALOG` ~11 城。
- `mustSeeEntryForCity("Lisbon") === null` → 无 seed、无 boost、无 diversity 输入；仅泛搜 QLP。
- 里斯本 A/B 探针（2026-08-22）：Arm A 种子 **pass**，但 knowledge 结论写成「ship Arm A catalog」——把实验手段写成产品方向。
- ADR-040 D8 用「西安池头」作本批硬门禁 → **用表内城验收巩固坏机制**。

### 第三次（must-see 簇 + dedup + name-split + district-filter，2026-08-23）

- `must-see-coverage.ts` `HARD_MUST_SEE_CLUSTERS = ["terracotta","dayan","wall"]` 西安簇名单 + `ensureHardMustSeeCoverage` 自动注入——表面通用，实际只对西安生效，对非西安静默失效。
- `discover-dedupe.ts` `attractionClusterKey` 西安地标正则 + `DIVERSITY_CLUSTER_ORDER` 优先序。
- `place-filters.ts:19` `LANDMARK_DASH_FRAGMENT` 西安地标后缀正则。
- `itinerary-planner.ts:720` `FAR_DISTRICT_HINT` 西安远郊行政区正则。
- 为何 ADR-042 没拦住：原措辞钉 `discover-must-see.ts` 文件名，没钉「源码任何城市 POI 知识」原则；新硬编码在 dedup/coverage/name-split/district-filter 名目下进别的文件，逃出视野；禁增长没清存量；无全局守卫测试。

### 三条「真·任意目的地」路径当时都不通

1. 扩 CATALOG — 可扩展性差（本次否定）。  
2. L1 LLM 写 query — 探针有、生产禁（performance / ADR-038）。  
3. 供应商热门榜 API — **从未接到** `searchCandidatePools`。

## 为什么会重犯（过程根因）

1. **教训未升格为硬拒绝**  
   第一次修的是「谁来选 provider」，没有写成：「禁止用城表承载目的地百科当质量主策略」。下次换场景（must-see）就当作新问题重新发明表。

2. **局部 AC 绑架设计**  
   「西安必须出兵马俑」是真需求；最快绿线是写西安种子。ADR/DoD 只钉锚点城 → 全球能力从未被定义。

3. **探针结论被误读**  
   「有种子就过 must_see」≠「种子表是架构」。Lisbon knowledge 的 ship catalog 句直接推动重犯。

4. **约束被误读为唯一解**  
   「L1 不接 LLM」被当成「只能硬编码」。漏掉第三解：供应商热门/外置 pack。

5. **测试与故事奖励错误行为**  
   单测断言 CATALOG 命中西安会绿；加城故事好估点；接 popularity API 难估 → 代理人/排期偏好扩表。

6. **同批 ADR 自我加固**  
   D8 把 ADR-038 机制当本批 DoD，未先质问机制是否可扩展 → 文档闭环，现场一问里斯本即破功。

## Lesson / guidance

**审稿一问（默认拒绝）：**  
「这个方案是否把某个城市的专名/必去/菜式写进仓库源码或调用方，并指望靠加行覆盖世界？」→ 是则引用 [ADR-042](../../adr/ADR-042-no-city-encyclopedia-in-source.md)，改目的地无关信号或外置数据。**2026-08-23 升级：** 范围从「禁 CATALOG 增长」扩到「源码禁止任何城市 POI 知识」（含去重簇、后缀正则、行政区过滤）；新增 `tests/no-city-hardcode.test.ts` 守卫测试把原则钉成 CI 闸，防第四次。必去知识改由 LLM 从候选池推断（`inferMustSeeFromPool`，prompt 无城市名），对所有城市通用。

**验收一问：**  
「非表内目的地（如 Lisbon）是否仍具备同等 must-see/热门能力？」→ 否则不得宣称 L1 质量完成。

**探针一问：**  
「Arm 是在证明机制，还是在证明可以扩表？」→ 只许把机制写进 ADR，不许把「加一行种子」写成 Verdict。

## Links

- [ADR-042](../../adr/ADR-042-no-city-encyclopedia-in-source.md)
- [ADR-038](../../adr/ADR-038-discover-places-quality.md)（机制被部分废止）
- [discover-lisbon-ab-probe.md](./discover-lisbon-ab-probe.md)
- [mvp3a-provider-auto-selection.md](../ops/mvp3a-provider-auto-selection.md)
