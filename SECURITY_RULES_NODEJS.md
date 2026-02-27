# SECURITY_RULES_NODEJS.md — Node.js security rules for AI-assisted coding

This file is **canonical for Node runtime/service concerns** (servers, jobs, scripts, libraries).
For Next.js framework concerns, also follow `SECURITY_RULES_NEXTJS.md`.

---

## 1) Secrets & credentials (non-negotiable)
- Never commit secrets in git (`.env`, configs, scripts).
- Never log secrets, tokens, cookies, passwords, connection strings.
- Use server-only environment variables or a secret manager.
- Separate credentials per environment; rotate if exposure is suspected.

---

## 2) Input validation & safe parsing
- Treat request bodies, query params, headers, cookies, and webhooks as untrusted.
- Validate at boundaries with schemas (Zod/Joi/etc.).
- Enforce limits (size, length, bounds, array sizes).
- Avoid ReDoS-prone regex; avoid parsing huge payloads.

---

## 3) Injection & unsafe execution
- SQL/NoSQL injection: always parameterize queries / use ORM builders.
- Command injection:
  - avoid `exec/spawn` where possible
  - never pass user-controlled strings into shell commands
- Avoid dynamic code execution:
  - no `eval`, `new Function`, dynamic `require` from user inputs

---

## 4) SSRF and outbound networking
If a user can influence a URL:
- Prefer allowlisting domains/hosts.
- Block private IP ranges and metadata endpoints.
- Enforce timeouts/abort signals and response size limits.

---

## 5) File system safety
- Prevent path traversal:
  - normalize paths
  - enforce a fixed base directory
  - reject `..` and absolute paths
- Store uploads safely; do not execute uploaded content.
- Avoid reading arbitrary file paths from user inputs.

---

## 6) AuthZ and multi-tenant safety (services)
- Enforce object-level authorization checks on the server.
- Never trust client-provided roles/tenant IDs without verification.
- Add audit logs for privileged actions (minimal data, redacted).

---

## 7) Dependencies & supply chain
- Minimize dependencies; document why new ones are added.
- Pin versions via lockfile.
- No “curl | bash”.
- Prefer reputable maintainers; review high-risk deps (auth/crypto/parsers).

---

## 8) Logging, monitoring, incident readiness
- Redact secrets and PII in logs.
- Use structured logs where possible.
- On suspected key exposure:
  - rotate keys
  - invalidate sessions/tokens
  - review access logs

---

## 9) Quality gates
Before finalizing:
- `npm run lint`
- `npm test` (if present)
- `npm run build`
