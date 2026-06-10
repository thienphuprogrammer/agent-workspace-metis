---
mode: agent
description: Fix a bug using CodeGraph intelligence — traces root cause via callers graph before any edit
---

Fix the following bug using the CodeGraph-first workflow.

**Bug description:** ${input:description}

## Required steps (do not skip any)

1. Call `codegraph_explore` with keywords from the bug description — REQUIRED FIRST, no exceptions.
2. Call `codegraph_callers` on the primary suspect symbol to build the call chain.
3. Call `codegraph_node` on the broken symbol to read its source. Do NOT use the Read tool on indexed files.
4. State the root cause in one sentence before touching any code.
5. Call `codegraph_impact` on every symbol you plan to change — REQUIRED before any edit.
6. Apply the minimal patch that fixes the root cause. No refactoring, no renames, no extra logging.
7. Run the relevant test. Write a regression test first if none exists.

## Output format

**Context:** [symbols found by explore + callers chain]
**Analysis:** [root cause — exact symbol name, line number, broken invariant]
**Action:** [diff of patch applied]
**Verification:** [exact test command + expected output]
