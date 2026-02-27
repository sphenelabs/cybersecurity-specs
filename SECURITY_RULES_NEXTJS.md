# SECURITY_RULES_NEXTJS.md — Next.js security rules for AI-assisted coding

This file is **canonical for Next.js-only concerns** (React + App Router + Server Actions + Route Handlers).
For Node runtime/service concerns, also follow `SECURITY_RULES_NODEJS.md`.

---

## 1) Client vs server boundaries (non-negotiable)
- Anything that can reach a **Client Component** is **public**.
- Never access secrets in code that can be bundled client-side.
- Never store secrets in `NEXT_PUBLIC_*` env vars. Those are public by design.
- Avoid importing server-only modules from Client Components (can leak code/config).

✅ Allowed:
- `process.env.*` in Route Handlers / Server Actions / server utilities.
- Calling server endpoints from the client without exposing secrets.

❌ Not allowed:
- API keys in React components
- Secrets in `NEXT_PUBLIC_*`
- Copying secrets into logs or error messages visible in the browser

---

## 2) Route Handlers (`app/api/*`) and Server Actions
- Validate input at the boundary (body/query/params) with a schema (Zod recommended).
- Enforce **authn + object-level authz** on every privileged action.
- Return safe error messages to the client; log detailed errors server-side (redacted).
- Apply rate limits on:
  - auth endpoints (login/signup/reset)
  - webhooks
  - expensive endpoints (LLM calls, exports, searches)

---

## 3) Rendering and XSS
- Avoid `dangerouslySetInnerHTML`.
- If rendering user-provided HTML/markdown is required:
  - sanitize with a well-reviewed sanitizer
  - restrict allowed tags/attributes
  - document why HTML rendering is necessary
- Never build HTML with string concatenation from untrusted data.

---

## 4) Cookies, sessions, and CSRF (browser context)
- Use secure cookies:
  - `HttpOnly` for session cookies
  - `Secure` in production
  - `SameSite=Lax/Strict` as appropriate
- If using cookie-based auth and you have state-changing routes:
  - protect with CSRF tokens and/or same-site strategy
  - prefer POST-only mutations

---

## 5) Security headers
- Use CSP where feasible (especially if you render any HTML).
- Set sensible defaults (e.g., `X-Content-Type-Options: nosniff`).
- Prefer centralized header setup via Next.js middleware/config.

---

## 6) File uploads and downloads (Next.js routes)
- Validate content type, size, and extension.
- Store uploads outside web root; serve via controlled routes.
- Require authz checks for downloads; prevent IDOR.

---

## 7) Next.js environment & build safety
- Do not expose internal stack traces to users.
- Avoid shipping dev/debug endpoints in production.
- Keep secrets out of:
  - client bundles
  - `public/`
  - source maps (treat as potentially accessible)

---

## 8) LLM / RAG in Next.js apps (if applicable)
- Never let untrusted content override system/developer instructions (prompt injection).
- Tools must be allowlisted and schema-validated.
- Require explicit user confirmation for destructive/high-impact actions.
