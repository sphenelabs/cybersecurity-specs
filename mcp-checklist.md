# MCP Server and Client Security Audit Checklist

For each, go through the codes to verify yes or no. If yes, write a brief and technical justification. If no, suggest action points. Save all results in an md with AI model, commit version and scanned date in the file and the filename.

## MCP Server Security Audit Checklist

### 1) Transport & protocol compliance

* [ ] **HTTP-based MCP server** follows the MCP Authorization spec requirements (when auth is supported) ([Model Context Protocol][1])
* [ ] Server publishes **Protected Resource Metadata (PRM)** and challenges correctly (e.g., `401` with `WWW-Authenticate` guidance) ([Model Context Protocol][2])
* [ ] TLS everywhere (HTTPS), secure headers (HSTS where applicable), and safe CORS defaults (only required origins)

### 2) Authorization & token handling

* [ ] Authorization is enabled when handling **user data / sensitive operations** ([Model Context Protocol][2])
* [ ] Access tokens are validated on every request (issuer/audience/signature/expiry; no “trust by transport”)
* [ ] **Token passthrough is NOT used** (do not accept tokens not issued for your MCP server; do not forward client-provided tokens downstream) ([Model Context Protocol][3])

### 3) Consent & “confused deputy” (critical for MCP proxy servers)

* [ ] Per-`client_id` consent is enforced **before** initiating third-party auth flows (avoid consent-cookie bypass) ([Model Context Protocol][3])
* [ ] Consent cookies/sessions are hardened: `__Host-` prefix, `Secure`, `HttpOnly`, `SameSite=Lax`, signed or server-side sessions, **bound to `client_id`** ([Model Context Protocol][3])
* [ ] Redirect URI validation is strict (exact-match; no wildcard patterns) ([Model Context Protocol][3])
* [ ] OAuth `state` is cryptographically random, stored only **after user consent**, single-use, and validated ([Model Context Protocol][3])

### 4) Scope design & least privilege

* [ ] Scopes are **minimized** (avoid broad “*” scopes; use progressive scope requests) ([Model Context Protocol][3])
* [ ] High-impact operations require additional gating (step-up auth or explicit user confirmation) ([OWASP Cheat Sheet Series][4])

### 5) Session safety (especially SSE / resumable streams)

* [ ] If authorization is implemented: **verify all inbound requests**
* [ ] **Do not use sessions for authentication**
* [ ] Session IDs are secure, non-deterministic, rotated/expired, and bound to user identity in internal queues/streams ([Model Context Protocol][3])

### 6) Tooling controls (agentic risk)

* [ ] Tools follow **least privilege** (scoped permissions, allowlists, no wildcard access) ([OWASP Cheat Sheet Series][4])
* [ ] Tool inputs/outputs are schema-validated, size-limited, and treated as untrusted (tool output can be a prompt-injection vector) ([OWASP Cheat Sheet Series][5])
* [ ] High-risk tool calls have **human-in-the-loop** or policy approval ([OWASP Cheat Sheet Series][4])

### 7) Prompt-injection resilience

* [ ] Untrusted content is clearly separated from system/developer instructions
* [ ] Output is validated and never executed directly (structured outputs + deterministic validators) ([OWASP Cheat Sheet Series][5])

### 8) Secrets & data protection

* [ ] Secrets never appear in model-visible context; redact sensitive fields from tool outputs/logs ([OWASP Cheat Sheet Series][4])
* [ ] Tenant/user isolation for stored memory and context ([OWASP Cheat Sheet Series][4])

### 9) Logging, auditability, and abuse detection

* [ ] Audit logs record: user identity, client identity, scopes used, tool invocations, and sensitive actions (plus correlation IDs)
* [ ] Anomaly detection for unusual tool patterns, repeated auth failures, suspicious scope escalation ([OWASP Cheat Sheet Series][4])

**Evidence to collect (server):** PRM JSON + headers, auth server config, token validation code path, consent/redirect validation tests, scope catalog + mapping to endpoints, tool permission model, session/SSE design, redaction rules, audit log samples.

---

## MCP Client Security Audit Checklist (Host / Desktop / Gateway)

### 1) Authorization flow correctness (remote servers)

* [ ] Correctly handles `401` challenges and reads `resource_metadata` from `WWW-Authenticate` ([Model Context Protocol][2])
* [ ] Fetches and parses PRM; uses it to discover authorization servers and scopes ([Model Context Protocol][2])
* [ ] Performs authorization-server metadata discovery per spec (tries required well-known endpoints; handles issuer URLs with path components) ([Model Context Protocol][1])

### 2) SSRF protections during OAuth/metadata discovery (client-side critical)

* [ ] Enforce **HTTPS** for OAuth-related URLs in production (allow loopback exceptions only for dev) ([Model Context Protocol][3])
* [ ] Block requests to **private/reserved ranges** (incl. `169.254.169.254`) ([Model Context Protocol][3])
* [ ] Do not implement custom IP parsing/validation (encoding tricks) ([Model Context Protocol][3])
* [ ] Validate redirect targets (or disable auto-follow and validate each hop) ([Model Context Protocol][3])
* [ ] For server-side client deployments: use an **egress proxy / network policy** ([Model Context Protocol][3])

### 3) Local MCP server install / “one-click config” hardening

* [ ] Show **exact command without truncation** before execution ([Model Context Protocol][3])
* [ ] Clearly warn it executes code on the user machine; require explicit approval; allow cancel ([Model Context Protocol][3])
* [ ] Detect & highlight dangerous patterns (`sudo`, `rm -rf`, suspicious network/file access) ([Model Context Protocol][3])
* [ ] Run local servers sandboxed with minimal privileges; require explicit privilege grants (filesystem/network) ([Model Context Protocol][3])

### 4) Credential & secret handling

* [ ] Store tokens in OS keychain/secure storage; never log tokens
* [ ] Support token revocation / disconnect per server and per user

### 5) Tool safety & prompt injection defense (client as orchestrator)

* [ ] Treat **all server content/tool output as untrusted**; sanitize before adding to context ([OWASP Cheat Sheet Series][5])
* [ ] Enforce structured tool invocation (schema validation, bounds, no arbitrary code execution) ([OWASP Cheat Sheet Series][4])
* [ ] Human approval gates for high-impact actions ([OWASP Cheat Sheet Series][4])

### 6) Visibility & user control

* [ ] UI shows connected servers, granted scopes, and last-used times
* [ ] Clear prompts when scope elevation is required; avoid requesting broad scopes up front ([Model Context Protocol][3])

**Evidence to collect (client):** auth-challenge handling logs (redacted), SSRF test results (private IP/redirect/DNS rebinding cases), local install consent screenshots, sandbox configuration, token storage strategy, scope request UX, tool invocation validation rules, high-impact action approval flow.

[1]: https://modelcontextprotocol.io/specification/draft/basic/authorization "Authorization - Model Context Protocol"
[2]: https://modelcontextprotocol.io/docs/tutorials/security/authorization "Understanding Authorization in MCP - Model Context Protocol"
[3]: https://modelcontextprotocol.io/docs/tutorials/security/security_best_practices "Security Best Practices - Model Context Protocol"
[4]: https://cheatsheetseries.owasp.org/cheatsheets/AI_Agent_Security_Cheat_Sheet.html "AI Agent Security - OWASP Cheat Sheet Series"
[5]: https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html "LLM Prompt Injection Prevention - OWASP Cheat Sheet Series"
