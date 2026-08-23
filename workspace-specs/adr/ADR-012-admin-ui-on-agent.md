# ADR-012: Operator admin UI on the places-agent deployable

## Status
Accepted

## Context

places-agent gained an operator management web-app (public home, login, landing, admins, caller API keys, instructions, i18n). Alternatives were: a fourth product/stack (like what2eat); a separate `admin.places.agent-mate.ai` hostname; or folding operator UX into what2eat.

ADR-001 already says consumer UX stays on thin apps. ADR-009 already says three services / three images / three stacks. Neither said where operator UX lives.

## Decision

1. The **operator management UI ships inside the places-agent deployable**: same git repo (`1.places-agent/`), same image, same Portainer stack, same hostname `places.agent-mate.ai`.
2. This does **not** add a fourth service, image, or stack. ADR-009 stays in force.
3. The operator **browser** talks only **same-origin** to that host (HTML + admin APIs). Admin auth is a **session** (cookie), not a caller API key.
4. HTTP tools and MCP keep **caller API key** auth (ADR-003 / Feature 12). An admin session does not call map vendors by itself.
5. Map-vendor keys and OPENAI_CN stay in **env**. Caller API key **plaintext** is shown only at create/regenerate; stored hashes live in the agent’s data store. Resend is server-side for invite/reset mail.
6. places-agent still does **not** own what2eat / where2play consumer screens (ADR-001). It **does** own this operator UX.

## Rationale

- Caller keys and agent instructions are facts about this host; a fourth app would split the trust boundary and the hostname visitors already use.
- Same-origin admin BFF matches ADR-002 (no secrets in the browser).
- One process per image stays simpler than admin-nginx + agent-API as two containers for this MVP.

Rejected: fourth Portainer stack; admin on what2eat; putting map-vendor keys in the admin UI.

## Consequences

- places-agent image = operator web + admin BFF + HTTP API + MCP.
- Stack needs a durable store (volume or DB) for admin users and caller-key hashes, plus env for Resend, session secret, OPENAI_CN, and map-vendor keys.
- First deploy seeds the default admin (`admin` / `me@ethanhuang.com`); public register stays off.
- NPM/Cloudflare: one hostname → one container.

## Date
2026-08-17
