---
name: cg-design
description: System design and detail design documentation workflow using CodeGraph. Use when documenting a system area, producing a design doc, planning an architecture change, or creating a component diagram. Always calls codegraph_explore → files → impact to build the full picture before writing any documentation.
---

# cg-design

## HARD RULE — applies to ALL workflows

Before ANY analysis, edit, or answer:

1. **MUST** call `codegraph_explore` with keywords from the request — no exceptions.
2. **MUST NOT** call `Read` or `Grep` on indexed source files — use `codegraph_node` instead.
3. **MUST** call `codegraph_impact` before editing any symbol.
4. Only fall back to `Read`/`Grep` when codegraph returns "not initialized", for non-indexed files (configs, docs, lock files), or when a tool response starts with ⚠️ (staleness banner — that file is pending re-index).

## Trigger

Use this skill when the request contains: design, architecture, diagram, document, explain system, how does X work, detail design, DD, spec, ADR.

## Flow

### Step 1 — Explore the system area (REQUIRED FIRST)
Call `codegraph_explore` with the area name or key symbols.
Extract: primary classes/functions, their responsibilities, key relationships.

### Step 2 — Survey files
Call `codegraph_files` on the relevant directory.
Goal: get the complete file inventory — nothing is missed in the design.

### Step 3 — Map dependencies
Call `codegraph_callers` and `codegraph_callees` on the key symbols.
Goal: understand data flow (what feeds in) and outputs (what consumes this).

### Step 4 — Impact surface
Call `codegraph_impact` on the top-level symbol of the area.
Goal: document what depends on this system (blast radius = the system's consumers).

### Step 5 — Write design document
Structure:
- **Overview** — one paragraph: what this system does and why it exists.
- **Components** — table: component name | responsibility | key file.
- **Data Flow** — step-by-step: input → processing → output (use symbol names from CodeGraph).
- **Key Interfaces** — signatures of public API symbols (from `codegraph_node`).
- **Consumers** — who calls this system (from `codegraph_callers` / impact).
- **Constraints & Trade-offs** — known limitations, performance characteristics, design decisions with rationale.

## Output Template

**Context:** [explore + files + callers/callees results]
**Analysis:** [system boundaries, key design decisions identified]
**Action:** [design document written to docs/design/<area>.md]
**Verification:** [peer review checklist: does the doc match what codegraph_explore returns?]

## CodeGraph-specific notes

Existing design docs are in `docs/design/`. Mirror their structure.
For new language support design: reference `docs/design/dynamic-dispatch-coverage-playbook.md`.
For MCP tool design: reference `src/mcp/server-instructions.ts` as the single source of truth.
