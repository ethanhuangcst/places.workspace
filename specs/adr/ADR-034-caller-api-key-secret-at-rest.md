# ADR-034: Persist caller API secret for admin list Copy

## Status
Accepted

## Context
Operators need to copy a caller API key again from the Keys list (e.g. pasting into what2eat / where2play `.env`). Historically only `keyHash` + `prefix` were stored; plaintext was returned once at create/regenerate and never again. That blocked list Copy without regenerating (which breaks live callers).

Alternatives considered:
1. Copy prefix only — useless for Bearer auth.
2. Encrypt-at-rest with a new envelope env — safer, but requires new secrets and `protect-eng` confirmation.
3. Store plaintext `secret` alongside `keyHash` — simplest ops path, admin-session only.

## Decision
Store optional plaintext `CallerApiKey.secret` (nullable for pre-migration rows). Keep SHA-256 `keyHash` as the sole Bearer lookup. Admin `GET /api/admin/api-keys` returns `secret` for Copy; UI never expands the full secret in the table cell.

Legacy rows with `secret = null` disable Copy until Regenerate or re-Issue.

## Rationale
- Matches operator request (“API 进库”) without new env keys.
- Auth path unchanged (`authenticateCaller` still hashes Bearer → `keyHash`).
- Encrypt-at-rest deferred to a future story if threat model requires it.

## Consequences
- DB or admin-session compromise exposes all stored caller secrets — accept for private operator portal; rotate via Regenerate if leaked.
- Production keys created before migration must be regenerated (or re-issued) before Copy works.
- Product copy updated: secrets are no longer “show once only”; create panel still offers immediate copy.

## Date
2026-08-21
