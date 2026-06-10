# CodeGraph Agent Workspace

A set of Claude Code skills, agents, and commands that enforce **CodeGraph-first** tool usage across 6 development workflows.

## What's inside

| Layer | Files | Purpose |
|---|---|---|
| Skills | `skills/cg-*/SKILL.md` | Workflow playbooks — CodeGraph-first rule + step-by-step flow |
| Agents | `agents/*.md` | Orchestrator gateway + 4 specialist agents |
| Commands | `commands/cg-*.md` | Slash command entry points |
| Cursor rules | `cursor/rules/*.mdc` | Same 6 workflows for Cursor |
| opencode | `opencode/AGENTS.md` | All 6 workflows as opencode system prompt |
| Copilot prompts | `copilot/prompts/*.prompt.md` | All 6 workflows as Copilot slash commands |

## Install into your project

### Claude Code

```bash
# From the codegraph repo root:
cp -r workspace-agent/skills/*   /path/to/yourproject/.claude/skills/
cp -r workspace-agent/agents/*   /path/to/yourproject/.claude/agents/
cp -r workspace-agent/commands/* /path/to/yourproject/.claude/commands/
```

Your project must have CodeGraph initialized (`codegraph init -i`) and the MCP server configured.

### Cursor

```bash
mkdir -p .cursor/rules
cp -r workspace-agent/cursor/rules/* .cursor/rules/
```

Rules trigger automatically when Cursor detects the workflow intent from your chat (`alwaysApply: false`).

### opencode

```bash
# Project-level (this project only):
cp workspace-agent/opencode/AGENTS.md ./

# Global (all projects):
cp workspace-agent/opencode/AGENTS.md ~/.config/opencode/
```

### GitHub Copilot

```bash
mkdir -p .github/prompts
cp -r workspace-agent/copilot/prompts/* .github/prompts/
```

Usage in Copilot Chat: `/cg-bugfix login token expires too early`

## What ports across agents

| Layer | Claude Code | Cursor | opencode | Copilot |
|---|---|---|---|---|
| CodeGraph-first HARD RULE | ✅ skills | ✅ rules | ✅ AGENTS.md | ✅ prompts |
| 6 workflow playbooks | ✅ skills | ✅ rules | ✅ AGENTS.md | ✅ prompts |
| Orchestrator gateway routing | ✅ agent | ❌ | ❌ | ❌ |
| Slash command entry points | ✅ commands | ❌ | ❌ | ✅ prompts |

Agents and commands are Claude Code–specific. The CodeGraph-first enforcement and all 6 workflow playbooks port to all four.

## Commands

| Command | What it does |
|---|---|
| `/cg-bugfix <description>` | Fix a bug using CodeGraph root-cause trace |
| `/cg-feature <description>` | Implement a feature with CodeGraph context |
| `/cg-review <symbol>` | Code review with blast-radius impact analysis |
| `/cg-test <symbol>` | Generate tests for a symbol's uncovered paths |
| `/cg-security <component>` | OWASP security audit via CodeGraph entry-point trace |
| `/cg-design <area>` | Produce a design doc from CodeGraph system map |

## How enforcement works

Every command invokes the `codegraph` orchestrator agent, which **always** calls `codegraph_explore` before routing to a specialist. Specialists re-enforce the rule via their skill playbooks. Three layers — no path through the system skips CodeGraph.

## Updating the CodeGraph-first rule

Edit `skills/cg-core/SKILL.md` first (the canonical source), then copy the HARD RULE block into the top of each workflow skill.
# agent-workspace-metis
