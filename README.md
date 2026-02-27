# AI Security Rule Kit (Split)

This kit splits security rules into:
- `SECURITY_RULES_NEXTJS.md` (Next.js framework + browser/client concerns)
- `SECURITY_RULES_NODEJS.md` (Node.js runtime/service concerns)

## Recommended usage
Treat these as canonical and reference them from your AI tool entrypoints.

## Entrypoints for different AI tools

### OpenAI Codex / OpenCode (repo root)
Create `AGENTS.md`:
```md
Follow `SECURITY_RULES_NEXTJS.md` and `SECURITY_RULES_NODEJS.md`.
If conflict exists, these two files win.
```

### Claude Code (repo root)
Create `CLAUDE.md`:
```md
Follow `SECURITY_RULES_NEXTJS.md` and `SECURITY_RULES_NODEJS.md`.
Run: npm run lint && npm test (if present) && npm run build
```

### GitHub Copilot
Create `.github/copilot-instructions.md`:
```md
Follow `SECURITY_RULES_NEXTJS.md` and `SECURITY_RULES_NODEJS.md`.
Non-negotiable: no secrets in client/NEXT_PUBLIC_*, validate inputs, enforce authz, prevent SSRF.
```

### Optional Cursor
Create `.cursor/rules/00-global-security.mdc` that points to these files.
