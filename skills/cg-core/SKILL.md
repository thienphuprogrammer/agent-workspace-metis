---
name: cg-core
description: CodeGraph-first contract and tool cheat-sheet. This is the shared base embedded at the top of every cg-* skill. Call this skill directly to review the CodeGraph tool selection rules.
---

# CodeGraph-First Contract

## HARD RULE — applies to ALL workflows

Before ANY analysis, edit, or answer:

1. **MUST** call `codegraph_explore` with keywords from the request — no exceptions.
2. **MUST NOT** call `Read` or `Grep` on indexed source files — use `codegraph_node` instead.
3. **MUST** call `codegraph_impact` before editing any symbol.
4. Only fall back to `Read`/`Grep` when codegraph returns "not initialized", for non-indexed files (configs, docs, lock files), or when a tool response starts with ⚠️ (staleness banner — that file is pending re-index).

## Tool Selection Cheat-Sheet

| Intent | Tool |
|---|---|
| Understand a flow / find relevant symbols | `codegraph_explore` |
| Find where a named symbol is defined | `codegraph_search` |
| Read one symbol's source + who calls it | `codegraph_node` |
| Read a source file (replaces the Read tool) | `codegraph_node` with file path, no symbol |
| Who calls function X? | `codegraph_callers` |
| What does function X call? | `codegraph_callees` |
| What breaks if I change X? | `codegraph_impact` |
| List files in a directory | `codegraph_files` |
| Is the index healthy / initialized? | `codegraph_status` |

## Output Format

Every response MUST use these four sections:

- **Context** — CodeGraph findings (symbols found, call chains, relevant files)
- **Analysis** — root cause / design decision / coverage gap
- **Action** — code changes / plan / test cases
- **Verification** — command to run + expected output
