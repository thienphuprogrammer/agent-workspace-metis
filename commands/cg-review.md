---
description: Code review with blast-radius analysis — CodeGraph impact → checklist → verdict. Usage: /cg-review <symbol or description>
---

Route this request through the **codegraph** orchestrator with intent signal "review impact".

User request: $ARGUMENTS

If $ARGUMENTS is empty, call `codegraph_status`, then ask which symbols or files to review.

The orchestrator will:
1. Run `codegraph_explore` on the changed symbol names.
2. Route to the `cg-reviewer` specialist with the pre-fetched context.
3. `cg-reviewer` will apply the cg-review playbook: explore → impact → callers → checklist → verdict per symbol.

Do not answer directly. Invoke the codegraph orchestrator agent.
