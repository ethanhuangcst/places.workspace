# Knowledge Base

Reusable research conclusions, ops lessons, and domain notes (not code truth).

| Layer | Location |
| --- | --- |
| Product requirements | [`../1.req-specs.md`](../1.req-specs.md) (+ future app req docs) |
| Architecture decisions | [`../2.architecture.md`](../2.architecture.md), [`../adr/`](../adr/) |
| Deployment (release-bot) | [`../6.deployment-plan.md`](../6.deployment-plan.md) |
| Knowledge (this tree) | Consolidated handbook + probe matrix |

## Canonical docs

| Doc | Topic | Updated |
| --- | --- | --- |
| [handbook.md](./handbook.md) | **All** consolidated knowledge (naming, axes, providers, LLM, deploy, workspace) | 2026-08-20 |
| [maps/places-capabilities.md](./maps/places-capabilities.md) | AMAP / Google / Tripadvisor capability probe matrix | 2026-08-20 |
| [maps/price-level-live.md](./maps/price-level-live.md) | Live `price_level` / `price_per_person` coverage on search cards | 2026-08-20 |
| [maps/vendor-adapters.md](./maps/vendor-adapters.md) | AMAP lng,lat/GCJ-02; Google direct+MCP; Open-Meteo WMO; Tripadvisor enrich; **Taiwan 排除 AMAP** (ADR-026) | 2026-08-20 |
| [maps/cjk-region-detection-pitfalls.md](./maps/cjk-region-detection-pitfalls.md) | CJK 文本 ≠ 中国大陆；Geocode-first 替代 CJK 占比规则 (ADR-030) | 2026-08-20 |
| [llm/quanzil-gateway.md](./llm/quanzil-gateway.md) | Quanzil via `openai` SDK; not `api.openai.com` | 2026-08-17 |
| [i18n/hk-tw-output.md](./i18n/hk-tw-output.md) | HK vs TW three-layer output, glossary, Google `languageCode`, Open-Meteo weather codes | 2026-08-17 |
| [agent/places-agent-loop.md](./agent/places-agent-loop.md) | Tool loop; six HTTP+MCP tools; chat/enrich HTTP-only; **provider auto-selection** (ADR-026) | 2026-08-20 |
| [ui/agent-mate-admin-visual.md](./ui/agent-mate-admin-visual.md) | kb 性冷淡 operator chrome; integration guide table and hairlines | 2026-08-19 |
| [ui/what2eat-consumer-chrome.md](./ui/what2eat-consumer-chrome.md) | what2eat header Decide→Saved→Profile; Profile CJK 用户档/用戶檔 | 2026-08-21 |
| [ops/places-agent-next-runtime.md](./ops/places-agent-next-runtime.md) | Next 16 custom server, Edge middleware, Playwright waits, Prisma env | 2026-08-17 |
| [kb-ingest/README.md](./kb-ingest/README.md) | What is already in kb vs propose-only gaps | 2026-08-17 |
| [ops/yecao3-places-release.md](./ops/yecao3-places-release.md) | 野草云3 deltas vs kb-agent (one container, Postgres, ports) | 2026-08-20 |
| [ops/places-agent-admin-invite-dev.md](./ops/places-agent-admin-invite-dev.md) | Cross-machine invite testing; LAN dev origins; POST-not-GET gate | 2026-08-18 |
| [web-app-development/README.md](./web-app-development/README.md) | **Consolidated** Next/auth/E2E/mail lessons from MVP-1; KB ingest manifest | 2026-08-18 |
| [web-app-development/lessons-from-places-agent-mvp1.md](./web-app-development/lessons-from-places-agent-mvp1.md) | Full consolidated body (auth, Next 16, Playwright, coverage/ESLint, HTTP TC-H, MVP-2 close, **MVP-3a: server stability + caller decoupling**) | 2026-08-20 |
| [web-app-development/what2eat-mvp3-lessons.md](./web-app-development/what2eat-mvp3-lessons.md) | what2eat MVP-3 chat, hydrate, history live E2E | 2026-08-20 |
| [web-app-development/what2eat-mvp4-lessons.md](./web-app-development/what2eat-mvp4-lessons.md) | what2eat MVP-4 sort, chat UX, price, drafts, panel size | 2026-08-20 |
| [web-app-development/what2eat-mvp4-followups.md](./web-app-development/what2eat-mvp4-followups.md) | MVP-4 follow-ups: chat timeout, provider strip, ADR-031, card-first | 2026-08-20 |
| [web-app-development/what2eat-decide-locale-draft.md](./web-app-development/what2eat-decide-locale-draft.md) | Decide form draft vs locale `router.refresh()` / profile overwrite | 2026-08-20 |
| [web-app-development/what2eat-chat-agent-timeout.md](./web-app-development/what2eat-chat-agent-timeout.md) | Chat 502 ≈ BFF timeout; use `PLACES_AGENT_CHAT_TIMEOUT_MS` ≥ 90s | 2026-08-20 |
| [web-app-development/what2eat-chat-provider-auto-select.md](./web-app-development/what2eat-chat-provider-auto-select.md) | Chat omits providers; empty AMAP → Google (ADR-031) | 2026-08-20 |
| [maps/google-photos-media-url.md](./maps/google-photos-media-url.md) | Google Places photo `media` URL shape / proxy notes | 2026-08-20 |
| [testing/vendor-live-vs-fixture.md](./testing/vendor-live-vs-fixture.md) | Live mode must not serve fixture; DoD honesty gate (ADR-021) | 2026-08-19 |
| [testing/quality-gate-audit-2026-08.md](./testing/quality-gate-audit-2026-08.md) | Ghost E2E tests, fixture-coupled assertions, geocode default trap, CJK heuristic overseas misdetection | 2026-08-20 |
| [testing/cache-isolation-between-tests.md](./testing/cache-isolation-between-tests.md) | 模块级单例缓存跨测试污染；必须 afterEach clearCache | 2026-08-21 |
| [ops/places-agent-local-daemon.md](./ops/places-agent-local-daemon.md) | macOS `make dev` vs `make up`; **nohup 不可靠 → start_new_session** (ADR-035); health 为准 | 2026-08-22 |
| [agent/mcp-client-integration.md](./agent/mcp-client-integration.md) | Cursor `/mcp` vs ChatBox `/sse`; remote MCP config format | 2026-08-19 |
| [ops/safari-secure-cookie-localhost.md](./ops/safari-secure-cookie-localhost.md) | Safari 拒绝 HTTP localhost Secure cookie；`loadEnvConfig` 加载 `.env.production` 陷阱 | 2026-08-20 |
| [ops/mvp3a-provider-auto-selection.md](./ops/mvp3a-provider-auto-selection.md) | 三区域 provider 自动选择；台湾排除；caller 解耦；SessionManager；可 ingest KB 清单 | 2026-08-20 |
| [agent/llm-itinerary-token-optimization.md](./agent/llm-itinerary-token-optimization.md) | Token 优化 + **AbortSignal 硬超时**、arrange 1280/0.35、BFF `arrange_timeout` | 2026-08-22 |
| [agent/prompt-assembler-pattern.md](./agent/prompt-assembler-pattern.md) | Prompt 组装模式：base + overlay 片段拼接，不用模板引擎 | 2026-08-21 |

Older topic files under `architecture/`, `llm/`, `maps/provider-selection-*`, `naming/`, `ops/` were merged into `handbook.md` and removed.
