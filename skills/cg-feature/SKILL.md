---
name: cg-feature
description: Feature implementation workflow using CodeGraph intelligence. Use when adding new functionality, building a new capability, or extending an existing system. Always calls codegraph_explore → callees → impact before writing any code.
---

# cg-feature

## HARD RULE — applies to ALL workflows

Before ANY analysis, edit, or answer:

1. **MUST** call `codegraph_explore` with keywords from the request — no exceptions.
2. **MUST NOT** call `Read` or `Grep` on indexed source files — use `codegraph_node` instead.
3. **MUST** call `codegraph_impact` before editing any symbol.
4. Only fall back to `Read`/`Grep` when codegraph returns "not initialized", for non-indexed files (configs, docs, lock files), or when a tool response starts with ⚠️ (staleness banner — that file is pending re-index).

## Trigger

Use this skill when the request contains: add, implement, build, create, new feature, extend, support, enable.

## Flow

### Step 1 — Explore existing context (REQUIRED FIRST)
Call `codegraph_explore` with the feature domain keywords.
Goal: understand what already exists, find the extension points.

### Step 2 — Map callees of insertion points
Call `codegraph_callees` on the symbols where new code will plug in.
Goal: understand what the insertion point already calls so the new code fits the pattern.

### Step 3 — Survey related files
Call `codegraph_files` on the relevant directory.
Goal: find where analogous features live (use them as templates).

### Step 4 — Design (write before coding)
Write a 2-3 sentence description: what the feature does, what it touches, what it returns.
Identify: new symbols to create, existing symbols to extend.

### Step 5 — Check impact on extension points
Call `codegraph_impact` on every existing symbol you will modify.
Review: who else calls this? Will adding a parameter / changing behavior break them?

### Step 6 — Implement
Write the new code following patterns from analogous symbols found in Step 3.
Add to existing files when possible; create new files only when the responsibility is clearly new.

### Step 7 — Write tests
Write tests before or immediately after implementation — not as an afterthought.

## Output Template

**Context:** [explore results — existing symbols, extension points, analogous features]
**Analysis:** [design decision — what to create, what to extend, why]
**Action:** [code written / diff]
**Verification:** [test command + expected output]

## CodeGraph-specific notes

When adding a new language: use the `/add-lang` skill instead — it has the full pipeline.
When extending MCP tools: extension point is `src/mcp/tools.ts`. Check `codegraph_callees("ToolHandler")`.
When adding a new framework resolver: extension point is `src/resolution/frameworks/`. Mirror an existing resolver file.
