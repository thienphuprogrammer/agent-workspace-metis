---
name: cg-security
description: Security vulnerability scan workflow using CodeGraph + OWASP checklist. Use when auditing code for security issues, checking input validation, tracing data flow from user input to sensitive operations, or reviewing auth/crypto code. Always calls codegraph_explore → callers → node to trace data flow before applying the OWASP checklist.
---

# cg-security

## HARD RULE — applies to ALL workflows

Before ANY analysis, edit, or answer:

1. **MUST** call `codegraph_explore` with keywords from the request — no exceptions.
2. **MUST NOT** call `Read` or `Grep` on indexed source files — use `codegraph_node` instead.
3. **MUST** call `codegraph_impact` before editing any symbol.
4. Only fall back to `Read`/`Grep` when codegraph returns "not initialized", for non-indexed files (configs, docs, lock files), or when a tool response starts with ⚠️ (staleness banner — that file is pending re-index).

## Trigger

Use this skill when the request contains: security, vulnerability, vuln, OWASP, injection, XSS, auth, authentication, authorization, sensitive data, input validation, sanitize.

## Flow

### Step 1 — Explore entry points (REQUIRED FIRST)
Call `codegraph_explore` with the feature or component being audited.
Extract: all symbols that receive external input (route handlers, CLI arg parsers, file readers, API endpoints).

### Step 2 — Trace callers to entry points
Call `codegraph_callers` on each entry point.
Goal: confirm the full surface area of externally reachable code.

### Step 3 — Read entry point source
Call `codegraph_node` on each entry point (`includeCode: true`).
Look for: unsanitized user input passed to: file system, shell, SQL, eval, serialization, path operations.

### Step 4 — Apply OWASP Top 10 checklist
For each entry point, check:
- [ ] **A01 Broken Access Control** — is authorization checked before the operation?
- [ ] **A02 Cryptographic Failures** — are secrets/tokens handled securely? No hardcoded values?
- [ ] **A03 Injection** — is user input sanitized before use in shell/SQL/path/eval?
- [ ] **A04 Insecure Design** — does the design assume trust that shouldn't exist?
- [ ] **A05 Security Misconfiguration** — are defaults safe? Debug modes off?
- [ ] **A06 Vulnerable Components** — any deps with known CVEs? (`npm audit`)
- [ ] **A07 Auth Failures** — are sessions/tokens validated correctly?
- [ ] **A08 Integrity Failures** — is deserialized data validated?
- [ ] **A09 Logging Failures** — are security events logged without leaking sensitive data?
- [ ] **A10 SSRF** — can user input cause server-side requests to internal systems?

### Step 5 — Report findings
For each finding: severity (Critical/High/Medium/Low), affected symbol + line, reproduction path, recommended fix.

## Output Template

**Context:** [entry points found, caller trace, source read]
**Analysis:** [OWASP checklist results — only items with findings]
**Action:** [specific fixes — code diff per finding]
**Verification:** [how to confirm each fix is effective]

## CodeGraph-specific notes

MCP server entry points: `src/mcp/tools.ts` — all tool handlers receive user-controlled `args`.
Key validation already in place: `validatePathWithinRoot`, `validateProjectPath`, `MAX_INPUT_LENGTH`, `MAX_PATH_LENGTH` in `src/mcp/tools.ts`.
CLI entry point: `src/bin/codegraph.ts`. Shell injection surface: any `spawn()` or `exec()` call.
Run `npm audit` for dependency CVEs.
