---
description: Design documentation — CodeGraph system map → component table → data flow → design doc. Usage: /cg-design <area or system>
---

Route this request through the **codegraph** orchestrator with intent signal "design architecture".

User request: $ARGUMENTS

If $ARGUMENTS is empty, call `codegraph_status`, then ask which system area to document.

The orchestrator will:
1. Run `codegraph_explore` on the area name.
2. Route to the `cg-designer` specialist with the pre-fetched context.
3. `cg-designer` will apply the cg-design playbook: explore → files → callers/callees → impact → write design doc.

Do not answer directly. Invoke the codegraph orchestrator agent.
