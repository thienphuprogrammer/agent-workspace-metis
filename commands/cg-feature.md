---
description: Feature implementation workflow — CodeGraph context → design → implement → test. Usage: /cg-feature <description>
---

Route this request through the **codegraph** orchestrator with intent signal "feature implement".

User request: $ARGUMENTS

If $ARGUMENTS is empty, call `codegraph_status` to load project context, then ask what feature to build.

The orchestrator will:
1. Run `codegraph_explore` on keywords from this request.
2. Route to the `cg-solver` specialist with the pre-fetched context.
3. `cg-solver` will apply the cg-feature playbook: explore → callees → design → impact check → implement → test.

Do not answer directly. Invoke the codegraph orchestrator agent.
