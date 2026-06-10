---
description: Test generation — CodeGraph callees → coverage gap → write tests. Usage: /cg-test <symbol or area>
---

Route this request through the **codegraph** orchestrator with intent signal "test coverage".

User request: $ARGUMENTS

If $ARGUMENTS is empty, call `codegraph_status`, then ask which symbol or area to test.

The orchestrator will:
1. Run `codegraph_explore` on the symbol name.
2. Route to the `cg-tester` specialist with the pre-fetched context.
3. `cg-tester` will apply the cg-test playbook: explore → callees → read branches → coverage gap → write tests.

Do not answer directly. Invoke the codegraph orchestrator agent.
