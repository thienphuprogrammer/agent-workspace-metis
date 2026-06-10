---
mode: agent
description: Review code with blast-radius analysis using CodeGraph — produces impact report and per-symbol verdict
---

Review the following code change using the CodeGraph blast-radius workflow.

**What to review:** ${input:description}

## Required steps (do not skip any)

1. Call `codegraph_explore` with the changed symbol names — REQUIRED FIRST, no exceptions.
2. Call `codegraph_impact` on every changed symbol. Record the blast radius.
3. For any symbol with impact radius > 5 callers: call `codegraph_callers` for the full chain.
4. Call `codegraph_node` on each changed symbol to read the diff context.
5. Apply checklist per changed symbol:
   - Does the change break any caller's assumptions?
   - Are callers in the blast radius tested?
   - Does the change touch a public/exported symbol?
   - Are there edge cases not covered by existing tests?
   - Does the change introduce a new dependency?
6. Classify each changed symbol: **Safe** / **Needs test** / **Breaking change** / **Needs discussion**.

## Output format

**Context:** [symbols changed + their impact radius from codegraph_impact]
**Analysis:** [per-symbol verdict with checklist results]
**Action:** [required changes before merge — specific items only]
**Verification:** [which tests to run + expected output]
