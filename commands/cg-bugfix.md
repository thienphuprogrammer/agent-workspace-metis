---
description: Bug fix workflow — CodeGraph lookup → root cause → minimal patch → verify. Usage: /cg-bugfix <description>
---

Route this request through the **codegraph** orchestrator with intent signal "bug fix".

User request: $ARGUMENTS

If $ARGUMENTS is empty, call `codegraph_status` to load project context, then ask the user what is broken.

The orchestrator will:
1. Run `codegraph_explore` on keywords from this request.
2. Route to the `cg-solver` specialist with the pre-fetched context.
3. `cg-solver` will apply the cg-bugfix playbook: explore → callers → root cause → impact check → patch → verify.

Do not answer directly. Invoke the codegraph orchestrator agent.
