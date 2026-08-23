# Web app development

Reusable lessons from building operator-facing admin web apps on Next.js (first evidence: **places-agent MVP-1**, 2026-08-18). Not product AC — engineering process, failure modes, and test gates.

Product requirements stay in app `agent-specs/` / `req-specs`. Binding architecture stays in `../adr/`.

## Index

| Doc | Topic | Updated |
| --- | --- | --- |
| [lessons-from-places-agent-mvp1.md](./lessons-from-places-agent-mvp1.md) | **Consolidated** auth forms, Next 16, E2E, HTTP TC-H, integration guide table | 2026-08-19 |
| [what2eat-mvp1-lessons.md](./what2eat-mvp1-lessons.md) | what2eat MVP-1 DoD: build, contract tests, E2E, CI | 2026-08-19 |
| [what2eat-mvp2-lessons.md](./what2eat-mvp2-lessons.md) | what2eat MVP-2 Decide/details/Saved live probe | 2026-08-19 |
| [where2play-mvp2-close.md](./where2play-mvp2-close.md) | where2play MVP-2 save loop + live E2E / OPENAI_CN base URL | 2026-08-23 |
| [where2play-chat-01-quanzil.md](./where2play-chat-01-quanzil.md) | Plan chat = BFF OPENAI_CN（ADR-036）；不转发 agent `/v1/chat` | 2026-08-22 |
| [where2play-plan-l2-quanzil.md](./where2play-plan-l2-quanzil.md) | Plan L2 = BFF OPENAI_CN（ADR-037）；as-built vs Mode H `plan-11` | 2026-08-22 |
| [cross-product-spec-drift.md](./cross-product-spec-drift.md) | agent/2play 文档漂移根因；as-built vs target（ADR-039） | 2026-08-23 |
| [what2eat-mvp3-lessons.md](./what2eat-mvp3-lessons.md) | what2eat MVP-3 chat/history/hydrate E2E failure modes | 2026-08-20 |
| [what2eat-mvp4-lessons.md](./what2eat-mvp4-lessons.md) | what2eat MVP-4 sort/chat UX/price/drafts/panel size | 2026-08-20 |
| [what2eat-mvp4-followups.md](./what2eat-mvp4-followups.md) | MVP-4 evening fixes: timeout, providers, ADR-031, card-first | 2026-08-20 |
| [what2eat-chat-agent-timeout.md](./what2eat-chat-agent-timeout.md) | Chat 502 at 25s = BFF timeout, not dead agent | 2026-08-20 |
| [what2eat-chat-provider-auto-select.md](./what2eat-chat-provider-auto-select.md) | Chat tools strip providers[]; same ADR-026 auto-select as Decide | 2026-08-20 |
| [what2eat-decide-locale-draft.md](./what2eat-decide-locale-draft.md) | Decide criteria draft vs locale refresh | 2026-08-20 |
| [../agent/mcp-client-integration.md](../agent/mcp-client-integration.md) | Cursor vs ChatBox MCP setup | 2026-08-19 |
| [../ops/places-agent-local-daemon.md](../ops/places-agent-local-daemon.md) | Local server daemon reliability | 2026-08-19 |

## Superseded / merged sources

These ops notes are folded into the consolidated doc; keep them as short pointers only:

| Old path | Status |
| --- | --- |
| [../ops/places-agent-admin-invite-dev.md](../ops/places-agent-admin-invite-dev.md) | Merged → § Cross-machine dev, § Auth forms |
| [../ops/places-agent-next-runtime.md](../ops/places-agent-next-runtime.md) | Merged → § Next.js App Router |

## KB ingest

Propose-only copies for **kb-agent** live under [`../kb-ingest/`](../kb-ingest/README.md). Writes are **propose → human confirm**. See the ingest manifest in [`../kb-ingest/web-app-development-2026-08-18.md`](../kb-ingest/web-app-development-2026-08-18.md).
