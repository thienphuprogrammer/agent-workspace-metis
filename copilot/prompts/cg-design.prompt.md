---
mode: agent
description: Produce a system design document using CodeGraph — maps components, data flow, and consumers before writing documentation
---

Produce a design document for the following system area using the CodeGraph design workflow.

**System area to document:** ${input:description}

## Required steps (do not skip any)

1. Call `codegraph_explore` with the area name — REQUIRED FIRST, no exceptions.
2. Call `codegraph_files` on the relevant directory for the complete file inventory.
3. Call `codegraph_callers` and `codegraph_callees` on key symbols to map data flow.
4. Call `codegraph_impact` on the top-level symbol to document consumers.
5. Write the design document with these sections:
   - **Overview** — one paragraph: what the system does and why it exists.
   - **Components** — table: component name | responsibility | key file.
   - **Data Flow** — step-by-step with symbol names from CodeGraph.
   - **Key Interfaces** — public API signatures from `codegraph_node`.
   - **Consumers** — who calls this system (from impact / callers).
   - **Constraints & Trade-offs** — limitations, design decisions, rationale.

## Output format

**Context:** [explore + files + callers/callees results]
**Analysis:** [system boundaries, key design decisions identified]
**Action:** [design document content]
**Verification:** [does the doc match what codegraph_explore returns?]
