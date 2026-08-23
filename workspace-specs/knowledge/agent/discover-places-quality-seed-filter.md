# Discover Places 质量 — 西安坏池 → seed + filter

## 现象
where2play 西安三日行程无兵马俑/城墙/大雁塔；池内多为博物馆分点、世博园分园、甚至「公司企业」园林；餐厅偏经开区连锁。

## 根因
`discover_places` 无 LLM，仅固定 QLP 泛词（博物馆/公园园林/餐厅），且未走 timed 已有的 `place-filters`。L2 只能从候选选点 → 合法但差。定向 `search_places「西安 兵马俑」` 同供应商可命中必去。Chatbox「更好」常为 MCP 失败后宿主先验，不是 search 成功。

## 做法（ADR-038）
保留 discover 门面、不接 LLM；热门城 must-see **search query** seed + 改进泛搜 + 过滤（公司企业/停车场/公交站）+ seed-token 排序后再 cap。

Seed 落地后若仍差：同主题近重复（大量「城墙-敌楼」）占满 cap 与 L2 前 N 候选 → 三日变「城墙碎片游」。需 **cluster 去重 + 多样性配额 + 碎片硬拒**，并放宽 L2 候选窗口与一日一主题约束。

## 勿做
删 discover 改裸 search_* 当 2play 主路径；用散文行程当 L1 验收标杆；默认给 Discover 接 LLM。
