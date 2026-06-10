---
mode: agent
description: Generate tests for a symbol using CodeGraph — maps callees and execution paths before writing test cases
---

Generate tests for the following symbol or area using the CodeGraph test workflow.

**Symbol or area to test:** ${input:description}

## Required steps (do not skip any)

1. Call `codegraph_explore` with the symbol name — REQUIRED FIRST, no exceptions.
2. Call `codegraph_callees` to map all side effects and dependencies.
3. Call `codegraph_node` on the symbol (`includeCode: true`) to read all branches.
4. List every execution path; mark which paths have no existing test.
5. For each uncovered path, write one test case:
   - Arrange: minimal setup using real data (no mocks unless crossing a network/DB boundary)
   - Act: call the symbol with the input that exercises the path
   - Assert: check the return value or side effect
6. Run tests — every new test must pass before adding the next.

## Output format

**Context:** [symbol explored, callees mapped, source read]
**Analysis:** [execution paths found, which are untested]
**Action:** [test cases written — one per uncovered path]
**Verification:** [exact test command + expected PASS output]
