---
name: cg-test
description: Test generation workflow using CodeGraph. Use when writing tests for a symbol, finding coverage gaps, or generating a test suite for a feature. Always calls codegraph_explore → callees → node to understand behavior before writing any test.
---

# cg-test

## HARD RULE — applies to ALL workflows

Before ANY analysis, edit, or answer:

1. **MUST** call `codegraph_explore` with keywords from the request — no exceptions.
2. **MUST NOT** call `Read` or `Grep` on indexed source files — use `codegraph_node` instead.
3. **MUST** call `codegraph_impact` before editing any symbol.
4. Only fall back to `Read`/`Grep` when codegraph returns "not initialized", for non-indexed files (configs, docs, lock files), or when a tool response starts with ⚠️ (staleness banner — that file is pending re-index).

## Trigger

Use this skill when the request contains: test, coverage, spec, unit test, integration test, test suite, write tests for.

## Flow

### Step 1 — Explore the symbol under test (REQUIRED FIRST)
Call `codegraph_explore` with the symbol name.
Extract: what the symbol does, its signature, its callers.

### Step 2 — Map callees (behavior surface)
Call `codegraph_callees` on the symbol.
Goal: every callee is a potential mock boundary or side effect to assert.

### Step 3 — Read symbol source
Call `codegraph_node` on the symbol (`includeCode: true`).
Extract: all branches (if/else, switch, error paths) — each branch needs at least one test case.

### Step 4 — Coverage gap analysis
List all execution paths from Step 3.
Mark which paths have no existing test (check `__tests__/` with `codegraph_files`).

### Step 5 — Write tests
For each uncovered path, write one test case:
- Arrange: minimal setup using real data (no mocks unless crossing a network/DB boundary)
- Act: call the symbol with the input that exercises the path
- Assert: check the return value or side effect

### Step 6 — Run and verify
Run tests. Every new test must pass. Fix any failure before adding the next test.

## Output Template

**Context:** [symbol explored, callees mapped, source read]
**Analysis:** [execution paths found, which are untested]
**Action:** [test cases written — one per uncovered path]
**Verification:** [exact test command + expected PASS output]

## CodeGraph-specific notes

Test files live in `__tests__/` mirroring `src/`. New tests use `vitest`.
Run: `npm test -- -t "<symbol or pattern>"` to run targeted tests.
Use real SQLite (no DB mocks) — see CLAUDE.md: "Tests create temp dirs with fs.mkdtempSync and clean up in afterEach."
For platform-specific tests: gate with `it.runIf(process.platform === 'win32')(...)`.
