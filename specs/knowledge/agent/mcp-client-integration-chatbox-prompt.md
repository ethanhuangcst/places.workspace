### ChatBox system prompt template (ADR-040 D7 #3 — **optional ops only**)

**产品验收不依赖此项。** 终端用户无需改 ChatBox system prompt。  
边界一次一问与按天排程由 **MCP intake 返回值 + tool description + arrange 默认 agent** 承担。

若运维自建助手想额外加强路由，可选用下方模板；普通用户请忽略。

```text
（可选）Prefer places-agent discover_places → arrange_day one day at a time. Ask only questions returned in intake.question.
```
