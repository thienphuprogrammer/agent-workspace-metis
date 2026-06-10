# CodeGraph Agent Instructions

You are working in a codebase indexed by CodeGraph MCP. The following rules and workflows apply to all tasks.

---

## HARD RULE — applies to ALL tasks

Before ANY analysis, edit, or answer:

1. **MUST** call `codegraph_explore` with keywords from the request — no exceptions.
2. **MUST NOT** call `Read` or `Grep` on indexed source files — use `codegraph_node` instead.
3. **MUST** call `codegraph_impact` before editing any symbol.
4. Only fall back to `Read`/`Grep` when codegraph returns "not initialized", for non-indexed files (configs, docs, lock files), or when a tool response starts with ⚠️ (staleness banner — that file is pending re-index).

## Tool Selection

| Intent | Tool |
|---|---|
| Understand a flow / find relevant symbols | `codegraph_explore` |
| Find where a named symbol is defined | `codegraph_search` |
| Read one symbol's source + who calls it | `codegraph_node` |
| Read a source file | `codegraph_node` with file path, no symbol |
| Who calls function X? | `codegraph_callers` |
| What does function X call? | `codegraph_callees` |
| What breaks if I change X? | `codegraph_impact` |
| List files in a directory | `codegraph_files` |
| Is the index healthy? | `codegraph_status` |

## Output Format

Every response MUST use four sections: **Context** / **Analysis** / **Action** / **Verification**

---

## Workflow: Bug Fix

**Trigger:** bug, fix, error, crash, broken, regression, unexpected behavior, exception

1. `codegraph_explore` keywords from the bug description — REQUIRED FIRST
2. `codegraph_callers` on the suspect symbol to build the call chain
3. `codegraph_node` on the broken symbol to read source (NOT the Read tool)
4. State root cause in one sentence before touching code
5. `codegraph_impact` on every symbol to change — REQUIRED before edit
6. Apply minimal patch — no refactoring, no renames, no extra logging
7. Run or write a regression test

---

## Workflow: Feature Implementation

**Trigger:** add, implement, build, create, new feature, extend, support

1. `codegraph_explore` feature domain keywords — REQUIRED FIRST
2. `codegraph_callees` on insertion points to understand existing patterns
3. `codegraph_files` on relevant directory to find analogous features
4. Write a 2-sentence design statement before coding
5. `codegraph_impact` on every existing symbol to modify — REQUIRED before edit
6. Implement following patterns from analogous symbols
7. Write tests immediately after implementation

---

## Workflow: Code Review

**Trigger:** review, check, impact, blast radius, safe to merge, PR, diff

1. `codegraph_explore` changed symbol names — REQUIRED FIRST
2. `codegraph_impact` on every changed symbol — record blast radius
3. `codegraph_callers` for symbols with impact > 5 callers
4. `codegraph_node` on each changed symbol
5. Checklist per symbol: breaks callers? tested? public API? edge cases? new dep?
6. Verdict: **Safe** / **Needs test** / **Breaking change** / **Needs discussion**

---

## Workflow: Test Generation

**Trigger:** test, coverage, spec, unit test, write tests for

1. `codegraph_explore` symbol name — REQUIRED FIRST
2. `codegraph_callees` to map all side effects / dependencies
3. `codegraph_node` on symbol to read all branches
4. List every execution path; mark which are untested
5. Write one test per uncovered path: Arrange → Act → Assert
6. Use real data (no mocks unless network/DB boundary)
7. Every new test must pass before adding the next

---

## Workflow: Security Audit

**Trigger:** security, vulnerability, OWASP, injection, auth, sanitize

1. `codegraph_explore` component being audited — REQUIRED FIRST
2. `codegraph_callers` on each entry point to confirm attack surface
3. `codegraph_node` on each entry point
4. OWASP Top 10: A01 Access Control, A02 Crypto, A03 Injection, A04 Design, A05 Misconfiguration, A06 Components, A07 Auth, A08 Integrity, A09 Logging, A10 SSRF
5. Per finding: severity, symbol + line, reproduction path, fix

---

## Workflow: Design Documentation

**Trigger:** design, architecture, document, explain system, detail design, ADR

1. `codegraph_explore` area name or key symbols — REQUIRED FIRST
2. `codegraph_files` on relevant directory for complete file inventory
3. `codegraph_callers` + `codegraph_callees` on key symbols for data flow
4. `codegraph_impact` on top-level symbol for consumer list
5. Write design doc: Overview | Components table | Data Flow | Key Interfaces | Consumers | Constraints
