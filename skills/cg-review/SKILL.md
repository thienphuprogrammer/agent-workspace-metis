---
name: cg-review
description: Code review workflow with blast-radius analysis using CodeGraph. Use when reviewing a PR, a diff, or asking "what does changing X break?". Always calls codegraph_explore → impact → callers to produce a complete blast-radius report.
---

# cg-review

## HARD RULE — applies to ALL workflows

Before ANY analysis, edit, or answer:

1. **MUST** call `codegraph_explore` with keywords from the request — no exceptions.
2. **MUST NOT** call `Read` or `Grep` on indexed source files — use `codegraph_node` instead.
3. **MUST** call `codegraph_impact` before editing any symbol.
4. Only fall back to `Read`/`Grep` when codegraph returns "not initialized", for non-indexed files (configs, docs, lock files), or when a tool response starts with ⚠️ (staleness banner — that file is pending re-index).

## Trigger

Use this skill when the request contains: review, check, impact, blast radius, what breaks, PR, diff, safe to merge.

## Flow

### Step 1 — Explore changed symbols (REQUIRED FIRST)
Call `codegraph_explore` with the names of the symbols being changed.
Extract: which layer they belong to, what they depend on.

### Step 2 — Blast radius
Call `codegraph_impact` on every changed symbol.
Produce: list of affected symbols grouped by layer (same file / same module / cross-module).

### Step 3 — Caller trace for high-risk symbols
For any symbol with impact radius > 5 callers: call `codegraph_callers` to see the full call chain.
Flag: any public API or entry point in the impact set.

### Step 4 — Read changed symbol source
Call `codegraph_node` on each changed symbol to read the actual diff context.

### Step 5 — Review checklist
For each changed symbol, check:
- [ ] Does the change break any caller's assumptions?
- [ ] Are all callers in the blast radius tested?
- [ ] Does the change touch a public API (exported symbol)?
- [ ] Are there edge cases not covered by existing tests?
- [ ] Does the change introduce a new dependency?

### Step 6 — Verdict
Classify each changed symbol: **Safe** / **Needs test** / **Breaking change** / **Needs discussion**.

## Output Template

**Context:** [symbols changed + their impact radius from codegraph_impact]
**Analysis:** [per-symbol verdict with checklist results]
**Action:** [required changes before merge — specific items only, no vague suggestions]
**Verification:** [which tests to run + expected output]

## CodeGraph-specific notes

For CodeGraph PRs: pay extra attention to changes in `src/extraction/` (may break other languages), `src/mcp/tools.ts` (MCP API surface), and `src/installer/` (breaks installs silently).
Always check `codegraph_impact` on any change to `NodeKind` or `EdgeKind` in `src/types.ts` — these are used everywhere.
