---
title: Real-agent refinement checklist (past failures → plan_trip)
type: ops-lesson
status: active
as_of: 2026-09-05
tags:
  - plan_trip
  - adr-050
  - real-agent-refactory
related:
  - ../../adr/ADR-050-where2play-no-product-llm.md
  - ../../1.places-agent/agent-specs/real-agent-refactory.md
---

# Real-agent 细化检查表（相对最新设计审过）

**Target：** places-agent 真智能体环；对外 `plan_trip` + `fetch_trip_details`；where2play **零产品 LLM**（ADR-050）。

| 相对最新设计 | 含义 |
| --- | --- |
| 仍适用 | 细化 `plan_trip` 时必须钉死 |
| 须改写 | 旧表述依赖「2play 持 LLM」或旧工具链，按 Target 改写 |
| MCP-only | 仅 ChatBox/Cursor；2play HTTP 无此问题 |

| # | 主题 | 曾犯的错（摘要） | 相对最新设计 | 细化时要钉 |
| --- | --- | --- | --- | --- |
| 1 | AMAP / GMAP | locale/CJK 逼 AMAP；2play 汉字双源；discover 扩源污染 stay **与景点池** | 仍适用（收紧） | **ADR-052**：省略 `providers[]`；禁 CJK 判区；大陆 AMAP-only 空再 Google；**废除 discover 扩源**；列表抄卡、详情同身份+locale（D9/D10） |
| 2 | 搜索语言 | 中文品牌漏拉丁；中英葡名对不上；HK≠TW languageCode | 仍适用 | 当地语+用户语+拉丁品牌展开；对池用 `native_id`；禁 OpenCC 当港台本地化 |
| 3 | 餐窗 / 排餐 | 骨架锁店；reuse；圆心用酒店；circle 400；5km bias | 仍适用 | 骨架只档；午餐 near 景点；搜环+haversine 5km；早到钉窗 |
| 4 | 酒店 / 起点卡 | Hills→澳门；全名搜 `cards[0]`→钟楼；只传 name | 仍适用（收紧） | **ADR-053**：选定落整卡；填站只抄；禁 `cards[0]`；过远丢坐标不当定位；token 不得进 stay |
| 5 | 锚点=城市 | 酒店 80km 滤空池仍 200 | 仍适用 | 滤池锚=城市；全住宿不得 `ready` |
| 6 | 城表硬编码 | CATALOG / 簇 / 正则第三次重犯 | 仍适用 | 禁城→POI 表；DoD 含非目录城 |
| 7 | eligible / 合称 | 名胜区进芯片 → make 502 | 仍适用 | 同一 eligible；对不上降级不整单 502 |
| 8 | 芯片与池一致 | 助手/ungrounded 名当 must_include | 仍适用（收紧） | 第 6 题只 fetch `must_see`；2play 无模型，防 BFF 空选项伪造 |
| 9 | CRS | lng,lat vs lat,lng；GCJ vs WGS | 仍适用 | 入池 WGS84；调 AMAP 再转 |
| 10 | 空结果降级 | AMAP 空未补 Google；与大陆双源并行 / discover 扩源混淆 | 仍适用（收紧） | **ADR-052 D4**：仅自动 AMAP-only 且该次 0 卡 → 一次 Google；显式列表不回退；**discover 不是例外** |
| 11 | Google bias | place bias 误用 5km | 仍适用 | 景点/酒店城市尺度；5km 仅餐 |
| 12 | ETA / legs | LLM 估时；宿主丢 legs | 仍适用；MCP 段保留 | filled 只认 Directions；Web 不回传腿；MCP 仍防丢字段 |
| 13 | 跨日重复 | N× 同 plan 无 exclude | 仍适用 | 环内 usedNames / native_id |
| 14 | 节奏 / 全住宿 | 轻松过满；全住宿当成功 | 仍适用 | 密度上限；错误分码 |
| 15 | 必去覆盖 | 注入块；theme 当 covered | 仍适用 | 硬失败+重试；不注入 |
| 16 | 日期 nullish | `date: null`；默认今天 | 仍适用 | nullish；缺日期 need_input |
| 17 | token / AbortSignal | SDK timeout 不硬停 | 仍适用 | AbortSignal；校验失败才重试 |
| 18 | MCP 宿主纪律 | 并行、问确认、编造行程 | MCP-only | 2play 是确定性 HTTP，无 host_instructions 问题 |
| 19 | 会话 / 大 JSON | session_invalid；回传 skeleton | 仍适用 | 瘦响应+fetch；禁宿主回传大包 |
| 20 | HTTP 真源 | 写信封当 UI；本地散文 | 须改写 | 删「本地 Qwen」威胁；保留 fetch-only + artifacts merge |
| 21 | 四卡时机 | 芯片/四卡过晚 | 仍适用 | 城市→芯片；日期→四卡；visa 内部 adapter |
| 22 | 两个规划脑 | 2play L2 + agent 双脑 | 须改写 | **禁止** 2play 持产品 LLM；一律 `plan_trip` |
| 23 | i18n | 硬编码英文；餐档当店名 | 仍适用 | key 文案；档 id i18n |
| 24 | 天气 | 当地图天气；无 language | 仍适用 | Open-Meteo + WMO 目录 |
| 25 | 照片 | slim 丢 photo；带 key media；高德 http 被 https 门剥光；fill 按店名再搜图 | 仍适用（收紧） | ADR-051：写卡解析；**D6 高德 CDN http→https**；Google `maxWidthPx=800`；芯片/餐填站；起点选定解析、fill 有指针只抄；fetch 只读；2play 不升协议 |
| 26 | Tripadvisor | 当 providers / MCP | 仍适用 | 不进规划环主路径 |
| 27 | live/fixture | live 夹具；default-HK | 仍适用 | live 无 key 则 skip |
| 28 | 测试诚实 | 鬼 E2E；误连真实 DB | 仍适用 | 行为断言；库隔离 |
| 29 | 本地 env | 继承 2play DATABASE_URL | 仍适用 | health + 本库 |
| 30 | 错误映射 | 一 key 盖多失败 | 仍适用 | needs_input / failed 分码 |
| 31 | tool description | 宿主跟描述走 | MCP-only / Web 改契约 | Web 读 need_input schema |
| 32 | 景点库时机 | 先库再止血 | 仍适用 | 库最后；非 CATALOG |
| 33 | 规格漂移 | Done≠消费；ADR-052 已决未执行 | 仍适用（重点变） | 2play Qwen vs ADR-050；discover 扩源 vs ADR-052（见 `adr-052-discover-expansion-drift`）；Feature 89 |
| 34 | 过度设计闸 | 叠机制仍失败 | 仍适用 | 先删后加 |
| 35 | what2eat 隔离 | 行程重构误收 2eat 工具或关 agent Qwen | 仍适用 | 2eat 仅 geocode / search_restaurants / get_place_details / chat；不 plan_trip；产品 LLM 另议；合入前 Decide+chat 回归 |

## 2play 无 LLM 后的 intake（补一条）

| 角色 | 做什么 |
| --- | --- |
| 2play UI/BFF | 展示 agent `need_input`；回传答案；`POST plan_trip`；fetch 芯片/四卡/骨架 |
| places-agent | 全部 LLM；验真；写 Trip |

规范正文：[`real-agent-refactory.md`](../../../1.places-agent/agent-specs/real-agent-refactory.md)。
