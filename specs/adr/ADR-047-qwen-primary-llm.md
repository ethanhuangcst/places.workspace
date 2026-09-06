# ADR-047: 主 LLM 改用 Qwen（阿里云百炼 compatible-mode）

## Status
Accepted（2026-09-03）

## Context
places-agent 与产品 BFF 的主聊天/结构化生成原先走 **OPENAI_CN**（`OPENAI_*` + `openai` SDK）。`gpt-5.4` 对 `make_itinerary` 骨架等长 JSON 调用不稳定（空 `days`、160s 超时、同输入结果漂移）。

决定将 **主 LLM** 切到阿里云百炼 **Qwen**，经 **OpenAI-compatible** 端点，避免换 SDK。

相关： [ADR-004](./ADR-004-quanzil-fixed-per-deployable.md)、[ADR-041](./ADR-041-openai-cn-replaces-quanzil.md)。ADR-041 的「对外称呼 OPENAI_CN / 变量名 `OPENAI_*`」对本故事 **被本 ADR 覆盖**（OPENAI_CN 降为备用网关）。

## Decision

### D1 — 主 LLM = Qwen

1. 三个可部署体（places-agent、what2eat BFF、where2play BFF）的 **主 chat / 结构化 JSON LLM** 为 **Qwen**。  
2. 默认聊天模型：`QWEN_CHAT_MODEL=qwen-plus`。  
3. 备用聊天模型（未自动切换前仅记录）：`QWEN_CHAT_MODEL_FALLBACK=qwen-flash / qwen-turbo`。  
4. 图像（若产品需要）：`QWEN_IMAGE_MODEL=qwen-image-2.0`。  
5. 客户端仍用 `openai` SDK（或等价 `fetch` `/chat/completions`），`baseURL` = `QWEN_BASE_URL`（compatible-mode `/v1`）。  
6. 不按搜索目的地切换模型（ADR-004 轴不变）。

### D2 — 环境变量

各部署体服务端 env（不写浏览器、不写死密钥）：

| Code | Role |
| --- | --- |
| `QWEN_API_KEY` | 百炼 API Key（操作者粘贴；文档不写值） |
| `QWEN_HOST` | 工作空间 host |
| `QWEN_BASE_URL` | OpenAI-compatible chat base（`…/compatible-mode/v1`） |
| `QWEN_NATIVE_BASE_URL` | 原生 REST（`…/api/v1`）；主路径不用 |
| `QWEN_WORKSPACE` | 工作空间 id |
| `QWEN_REGION` | 如 `cn-beijing` |
| `QWEN_CHAT_MODEL` | 默认 `qwen-plus` |
| `QWEN_CHAT_MODEL_FALLBACK` | `qwen-flash / qwen-turbo` |
| `QWEN_IMAGE_MODEL` | `qwen-image-2.0` |
| `QWEN_KEY_MGMT_SITE` | 百炼控制台（北京） |

本工作空间约定的 **host / URL 形态** 写在 [`3.tech-specs.md`](../3.tech-specs.md) 与 [`knowledge/llm/qwen-gateway.md`](../knowledge/llm/qwen-gateway.md)。密钥只在各仓 `.env.local` / Portainer。

### D3 — 解析优先级

运行时：`QWEN_API_KEY` 非空且非 `fixture` → 用 Qwen。否则回退 `OPENAI_*`（OPENAI_CN）。两者皆无 → fixture / 未配置。

**不得删除** 已有 `OPENAI_*` 行（`protect-eng`）。OPENAI_CN 保留为备用，不是主路径。

### D4 — 规格用语

现行需求/设计/技术规格写 **Qwen** + `QWEN_*`。历史 ADR / 探针里的 OPENAI_CN / Quanzil 保持当时事实，并指向本 ADR。

## Consequences

- 骨架与助手延迟/JSON 稳定性取决于百炼 `qwen-plus`，不再默认 `gpt-5.4`。  
- 须在三个 `.env.local` 填入 `QWEN_API_KEY` 后重启进程，主路径才会切到 Qwen。  
- 自动 fallback 到 `qwen-flash` / `qwen-turbo` 未在本 ADR 实现，仅预留变量。

## Date
2026-09-03
