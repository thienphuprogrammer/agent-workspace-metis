---
name: cg-bugfix
description: Bug fix workflow using CodeGraph intelligence. Use when the request involves fixing a bug, error, crash, unexpected behavior, or regression. Always calls codegraph_explore → callers → impact before any edit.
---

# cg-bugfix

## HARD RULE — applies to ALL workflows

Before ANY analysis, edit, or answer:

1. **MUST** call `codegraph_explore` with keywords from the request — no exceptions.
2. **MUST NOT** call `Read` or `Grep` on indexed source files — use `codegraph_node` instead.
3. **MUST** call `codegraph_impact` before editing any symbol.
4. Only fall back to `Read`/`Grep` when codegraph returns "not initialized", for non-indexed files (configs, docs, lock files), or when a tool response starts with ⚠️ (staleness banner — that file is pending re-index).

## Trigger

Use this skill when the request contains: bug, fix, error, crash, broken, regression, wrong behavior, unexpected output, exception, failure.

## Flow

### Step 1 — Explore (REQUIRED FIRST)
Call `codegraph_explore` with keywords describing the broken behavior.
Extract: entry point symbols, suspect files, relevant call sites.

### Step 2 — Trace callers
Call `codegraph_callers` on the primary suspect symbol.
Goal: build the call chain from user-facing code down to the broken symbol.

### Step 3 — Read suspect symbol
Call `codegraph_node` on the specific broken symbol (`includeCode: true`).
Do NOT use the `Read` tool on indexed files.

### Step 4 — Root cause
Identify: which symbol, which line, which invariant is violated, what input triggers it.
Write this as one sentence before touching any code.

### Step 5 — Check blast radius (REQUIRED before edit)
Call `codegraph_impact` on every symbol you plan to change.
Review: what else calls this? Will the fix break callers?

### Step 6 — Apply minimal patch
Edit only the lines that fix the root cause.
Do NOT refactor surrounding code. Do NOT rename variables. Do NOT add logging.

### Step 7 — Verify
Run the relevant test. If no targeted test exists, write a regression test first, then apply the fix.

## Output Template

**Context:** [symbols found by explore + callers chain — paste actual results]
**Analysis:** [root cause — exact symbol name, line number, broken invariant]
**Action:** [diff of patch applied]
**Verification:** [exact test command + expected output]

## CodeGraph-specific notes

When fixing extraction bugs: `codegraph_explore("python extractor parse")` or similar. Suspect files are in `src/extraction/languages/`. Run `npm test -- -t "<LanguageName>"` to verify.
When fixing resolution bugs: look in `src/resolution/`. Run `npm test -- -t "resolution"`.
When fixing MCP tool bugs: look in `src/mcp/tools.ts`. Run `npm test -- -t "mcp"`.
