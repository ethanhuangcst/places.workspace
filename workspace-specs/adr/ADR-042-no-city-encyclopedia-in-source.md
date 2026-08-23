# ADR-042: 禁止把目的地百科写进源码当质量策略（must-see / 搜词）

## Status
Accepted（部分 supersede [ADR-038](./ADR-038-discover-places-quality.md) 的「热门城白名单 CATALOG」机制；**保留** ADR-038 的：L1 不接 LLM、discover 门面、硬过滤/排序/去重）。**2026-08-23 Update：** 范围从「禁 CATALOG 增长」升级为「源码禁止任何城市 POI 知识」原则（见文末 Update 节）。

## Context

**第二次重犯同一类错误：**

| 次 | 场景 | 拙劣形态 | 后果 |
| --- | --- | --- | --- |
| **1** | 餐厅推荐 / provider 路由 | 调用方或 agent **按城/按区硬编码**地理策略（如 what2eat `providersForPin` 双源、fixture/解析里堆城市分支） | 北京搜到美国餐厅等；见 [ADR-026](./ADR-026-region-based-provider-auto-selection.md)、[ADR-030](./ADR-030-geocode-first-region-detection.md)、knowledge `mvp3a-provider-auto-selection` |
| **2** | `discover_places` must-see | `discover-must-see.ts` **按城硬编码** `CATALOG`（~11 国内城 query/token） | 里斯本等非表内城 **无** must-see 能力；探针 Arm A 证明种子有效却未也不该靠「再加一行里斯本」扩展产品 |

ADR-038 在西安压力下把「国内热门城白名单 seed」写成 Accepted；里斯本探针 knowledge 甚至写「ship Arm A catalog」。ADR-040 D8 又以「西安池头」作本批硬门禁——**用锚点城验收巩固了可扩展性极差的机制**，而非质疑机制本身。

根因不是「忘了合入里斯本种子」，而是：**把目的地理知识写进源码当 L1 质量策略**——不可扩展、不可维护，且与「任意目的地」产品承诺冲突。

## Decision

1. **禁止**将「按城市维护百科式 seed / 必去名单 / 本地菜名单」作为 **产品级、可扩展的** must-see（或同类 L1 质量）方案。  
   - **禁止**为过验收继续往 `CATALOG` 加城（含里斯本）当作关闭 D8 / ADR-038 的主路径。  
   - 现有 `CATALOG` 至多视为 **临时止血**，必须有替换故事与下线条件；**不得**再开「扩表白名单」故事当能力交付。

2. **必须**用**目的地无关**（或数据外置、非源码百科）的机制获得「当地热门 / 必去」信号。候选（实现另开故事，本 ADR 定方向）：  
   - 供应商 **热门/流行度** 能力（如 Google 流行度/附近 attractions 排序、Tripadvisor attractions nearby 等——以实探为准）；  
   - 或可配置/可发布的 **外部 pack**（非把地理百科嵌进 TS 源码）；  
   - **不**把「L1 接 LLM 写 query」当默认（仍守 performance / ADR-038 无 LLM L1），除非另 ADR 明确改判。

3. **验收反模式作废：**  
   - 「锚点城（西安）绿 = L1 must-see 完成」**不够**；至少再验一个 **非 CATALOG** 目的地（如 Lisbon）池头覆盖，或证明走了目的地无关热门路径。  
   - 单测只断言 `mustSeeQueriesForCity("西安").length > 0` **不能**充当产品 must-see 能力证明。

4. **过程门禁（防第三次）：** 凡提案含「按城 if/表驱动地理知识」提升搜索/推荐质量，**默认拒绝**；须先引用本 ADR，并说明为何不是又一次 CATALOG。餐厅侧同类（caller 硬编码区域路由、无限城市 exclusion 表当主策略）同审。

## Rationale

- **第一次（餐厅）**已证明：地理策略写在错误层 / 写成表，会在未见过的目的地静默失败，且测试易只覆盖表内城。  
- **第二次（discover）**在已有教训下仍被 ADR-038「先过西安」选中，因为：  
  - 探针证明「有种子就好」→ 被误读为「种子表是产品」；  
  - 「L1 不接 LLM」被误读为「只能硬编码」；  
  - DoD 只钉西安，绿色掩盖全球缺口。  
- 备选「继续扩 CATALOG」已被产品否定（不可扩展、不可维护）。  
- 备选「立刻 L1 LLM」与既有延迟/成本/可测性决策冲突，不作本 ADR 默认。

## Consequences

- ADR-038：机制条款（白名单 CATALOG）由本 ADR **部分废止为产品策略**；过滤/无 LLM/门面仍有效。  
- ADR-040 D8：改为「must-see **能力**过线」，不得用「扩表过西安」关闭；实现故事另开。  
- 代码：`discover-must-see.ts` 标记技术债；新故事优先供应商热门或外置 pack，而非加 Lisbon 行。  
- Knowledge：记录重犯原因与审稿清单（本回顾）。  
- 更新 `2.architecture.md` ADR 表。

## Update（2026-08-23）：范围从「文件」升级为「原则」+ 复盘第三次重犯

本 ADR 原措辞只钉了 `discover-must-see.ts` 的 `CATALOG` 增长，没钉「源码任何城市 POI 知识」这条原则。结果第三次重犯：城市硬编码从 CATALOG 之外的「别的门」重新进来。本节正式升级范围并记录过程失败。

### 范围升级

- **原范围（窄）：** 禁 `discover-must-see.ts` `CATALOG` 继续加城。
- **升级范围（原则）：** **源码禁止任何城市 POI 知识**——包括但不限于必去名单、本地菜 token、地标去重簇、地标名后缀正则、远郊行政区过滤。城市名只允许出现在：测试 fixture、`normalizeCityName` 的归一化映射（系统工具，非 POI 知识）、注释里的 ADR 引用。
- 「与目的地无关的热度包」仍是未决技术债；在此之前，源码不得为任何单城编码质量策略。

### 第三次重犯：五处硬编码从「别的门」进来

| # | 文件 | 硬编码形态 | 为何 ADR-042 原措辞没拦住 |
| --- | --- | --- | --- |
| 1 | `must-see-coverage.ts` | `HARD_MUST_SEE_CLUSTERS = ["terracotta","dayan","wall"]` 西安簇名单 | 不在 `discover-must-see.ts`，是 coverage 注入层 |
| 2 | `discover-dedupe.ts` | `attractionClusterKey` 西安地标正则（城墙/钟楼/兵马俑/大雁塔/华清/回民街）+ `DIVERSITY_CLUSTER_ORDER` | 是去重质量，被当成「dedup 改进」而非「城表」 |
| 3 | `place-filters.ts:19` | `LANDMARK_DASH_FRAGMENT` 西安地标后缀正则 | 是卡片过滤，被当成「name-split 质量」 |
| 4 | `itinerary-planner.ts:720` | `FAR_DISTRICT_HINT` 西安远郊行政区正则 | 是餐厅排序，被当成「district filter」 |
| 5 | `discover-must-see.ts` | `CATALOG` 西安/杭州种子词（原 ADR-042 已点名，但只禁增长、未清存量） | 原措辞只禁加城，不清旧债 |

根因：ADR-042 禁了**文件名**（`discover-must-see.ts`），没禁**原则**（源码任何城市 POI 知识）；禁了**增长**，没清**存量**；新硬编码在 dedup / coverage / name-split / district-filter 等不同名目下被加进**别的文件**，逃出原 ADR 视野。且无全局守卫测试把原则钉成 CI 闸。

### 补救（C3 精简包，本日落地）

1. **清存量：** 删除上述五处全部城市硬编码，回退到与目的地无关的通用逻辑（rating 排序、规范化名去重、通用后缀处理）。
2. **升原则：** 本节把 ADR-042 范围正式升级为「源码禁止任何城市 POI 知识」。
3. **钉 CI：** 新增 `tests/no-city-hardcode.test.ts` 守卫测试，grep 扫 `src/core` + `src/mcp` 源码，断言不出现城市 POI 知识字串（白名单：测试文件、归一化映射、ADR 引用注释）。防止第四次。
4. **必去知识搬家：** must-see 自动注入不再靠源码簇名单，改为 `discover-must-see-llm.ts` 的 `inferMustSeeFromPool`——discover 后让 LLM 从候选池选 ≤3 个公认必去，prompt 不含任何城市名，知识来自 LLM 权重（与目的地无关）。结果经「必须在池中」校验防幻觉，作为 `must_include` 种子走现有 P0 硬通道。对所有城市通用，里斯本/青岛/杭州不再静默失效。

### 未决技术债

- 「与目的地无关的热度包」（供应商流行度 / 外置 pack）仍是 ADR-042 既定方向的真正替代；LLM 推断必去是过渡方案，每程 discover 多一次 LLM 调用（~1-2s）。热度包落地后可下线 LLM 推断。

## Date
2026-08-23
