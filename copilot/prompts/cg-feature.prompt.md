---
mode: agent
description: Implement a new feature using CodeGraph intelligence — explores existing patterns and checks blast radius before writing code
---

Implement the following feature using the CodeGraph-first workflow.

**Feature description:** ${input:description}

## Required steps (do not skip any)

1. Call `codegraph_explore` with the feature domain keywords — REQUIRED FIRST, no exceptions.
2. Call `codegraph_callees` on the insertion points to understand existing patterns.
3. Call `codegraph_files` on the relevant directory to find analogous features to use as templates.
4. Write a 2-sentence design statement before writing any code.
5. Call `codegraph_impact` on every existing symbol you will modify — REQUIRED before any edit.
6. Implement following the patterns from analogous symbols found in step 3.
7. Write tests immediately after implementation.

## Output format

**Context:** [explore results — existing symbols, extension points, analogous features]
**Analysis:** [design decision — what to create, what to extend, why]
**Action:** [code written / diff]
**Verification:** [test command + expected output]
