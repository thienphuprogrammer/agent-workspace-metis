---
description: Security audit — CodeGraph entry-point trace → OWASP Top 10 checklist → findings report. Usage: /cg-security <component or area>
---

Route this request through the **codegraph** orchestrator with intent signal "security vulnerability".

User request: $ARGUMENTS

If $ARGUMENTS is empty, call `codegraph_status`, then ask which component or feature to audit.

The orchestrator will:
1. Run `codegraph_explore` on the component name.
2. Route to the `cg-tester` specialist with the pre-fetched context.
3. `cg-tester` will apply the cg-security playbook: explore entry points → callers → node source → OWASP checklist → findings.

Do not answer directly. Invoke the codegraph orchestrator agent.
