# ADR-038: discover_places 质量 — 无 LLM 的 seed + 过滤门面

## Status
Accepted（**机制部分被 [ADR-042](./ADR-042-no-city-encyclopedia-in-source.md) / [ADR-043](./ADR-043-chatbox-mcp-and-cross-product-closure.md) supersede：** 禁扩城表；L1 用 POPULARITY + 热门模板。本 ADR 仍约束 L1 不接 LLM、discover 门面与过滤）

## Context
where2play / MCP 依赖 places-agent `discover_places` 作为 L1 候选池（[ADR-037](./ADR-037-where2play-plan-l2-quanzil.md)）。L2 被硬约束只能从候选选点。实测西安等城：固定 QLP 泛词（博物馆/公园园林/餐厅）+ discover **未**复用 timed 的 `place-filters`，导致池中无兵马俑等必去点、出现「公司企业」噪声；定向 `search_places` 同一供应商却能命中必去。Chatbox「更好行程」常来自宿主先验或手写 query，不能证明应删掉 discover 或给 L1 接 LLM。

## Decision
1. **保留** `discover_places` 为编排后的行程候选门面；底层仍调 `search_places` / `search_restaurants`。
2. **L1 不接 LLM**（与 [performance.md](../../1.places-agent/agent-specs/performance.md) 三层一致）。质量靠：热门城 **must-see seed query**、改进泛搜、硬过滤、seed-token 优先排序，再 cap。
3. **国内热门城白名单**（北京、上海、西安、成都、杭州、广州、深圳、南京、重庆、武汉、哈尔滨等）提供定向 search query；非白名单城仅改进泛搜+过滤（不保证必去点）。
4. Seed 存 **query 字符串**，不存假 POI；结果必须来自地图供应商。
5. discover 与 timed 共用并加硬 [`place-filters`](../../1.places-agent/src/core/place-filters.ts)（拒绝公司企业/停车场/公交站等）。
6. **近重复治理（P0）：** 拒绝售票处/直通车/敌楼等碎片；按 landmark cluster 每类只留 1 主 POI；热门城保证兵马俑/大雁塔/城墙等多样性进池头后再 cap。
7. **不**用裸 `search_*` 替代 discover 作为 2play 主路径（避免各端复制 merge/cap/过滤）。下游 L2 截断（如 prompt 只取前 N 条）是另故事；本 ADR 要求池**头部**优先必去点且不堆同 cluster 碎片。where2play L2 扩大候选窗口并约束一日一主题/同区连游（仍不调 navigate）。

## Rationale
- 同原语下，discover 应系统性地做到「聪明调用方」：多 query + 过滤 + 排序。
- 无 LLM 保持秒级、可测、真 POI；西安回归用 mock/live 契约，不用散文行程当标杆。

## Consequences
- 实现：`discover-must-see` 冻结、discover 专用 job、`searchCandidatePools` filter+rank、**Google POPULARITY + 热门模板**（ADR-043）。
- 维护成本：禁止扩城表；非模板热门靠供应商排序。
- **关闭包：** [ADR-043](./ADR-043-chatbox-mcp-and-cross-product-closure.md)。

## Date
2026-08-22
