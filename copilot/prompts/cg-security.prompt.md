---
mode: agent
description: Security audit using CodeGraph and OWASP Top 10 — traces entry points and data flow before applying checklist
---

Audit the following component for security vulnerabilities using the CodeGraph OWASP workflow.

**Component or area to audit:** ${input:description}

## Required steps (do not skip any)

1. Call `codegraph_explore` with the component name — REQUIRED FIRST, no exceptions.
2. Call `codegraph_callers` on each entry point to confirm the full attack surface.
3. Call `codegraph_node` on each entry point to read source. Look for unsanitized user input passed to file system, shell, SQL, eval, serialization, or path operations.
4. Apply OWASP Top 10 checklist:
   - A01 Broken Access Control — authorization checked before operation?
   - A02 Cryptographic Failures — secrets/tokens handled securely?
   - A03 Injection — user input sanitized before shell/SQL/path/eval?
   - A04 Insecure Design — design assumes trust that shouldn't exist?
   - A05 Security Misconfiguration — defaults safe? debug modes off?
   - A06 Vulnerable Components — any deps with known CVEs?
   - A07 Auth Failures — sessions/tokens validated correctly?
   - A08 Integrity Failures — deserialized data validated?
   - A09 Logging Failures — security events logged without leaking data?
   - A10 SSRF — user input can cause server-side requests to internal systems?
5. For each finding: severity (Critical/High/Medium/Low), affected symbol + line, reproduction path, recommended fix.

## Output format

**Context:** [entry points found, caller trace, source read]
**Analysis:** [OWASP checklist results — only items with findings]
**Action:** [specific fixes — code diff per finding]
**Verification:** [how to confirm each fix is effective]
