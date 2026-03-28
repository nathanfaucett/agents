# Agents

This repository contains local agent definitions under `agents/` and local skills
under `skills/`.

## Local Agents

Current local agents:

- `code-qa-engineer`
- `devops-engineer`
- `principal-engineer`
- `researcher-engineer`
- `security-engineer`
- `ux-engineer`

Link all local agents into `~/.claude/agents`:

```bash
mkdir -p ~/.claude/agents
for f in agents/*.agent.md; do
  ln -sf "$PWD/$f" "$HOME/.claude/agents/$(basename "$f")"
done
```

## Local Skills

Current local skills:

- `agent-ready`: Analyze a codebase for gaps that hinder autonomous agents and provide prioritized recommendations.
- `change-review`: Run a structured, multi-lens review and synthesize actionable findings before merge.
- `poc`: Produce a focused proof-of-concept plan with feasibility assessment and minimal experiment artifacts.
- `subagents-tasks`: Convert PRDs/issues/todos into iterative task plans with explicit sub-agent workflow signals.
- `subagents-tasks-run`: Execute the next pending task from a plan and update workflow state files.

Install all local skills:

```bash
for d in skills/*/; do
  npx skills add "./${d%/}" --global --symlink -y
done
```

## Optional External Skills

```bash
npx skills add \
  mattpocock/skills/grill-me \
  mattpocock/skills/improve-codebase-architecture \
  mattpocock/skills/prd-to-issues \
  mattpocock/skills/tdd \
  mattpocock/skills/write-a-prd \
  --global -y
```

- [grill-me](https://github.com/mattpocock/skills): Deeply interview the user to stress-test a plan or design.
- [improve-codebase-architecture](https://github.com/mattpocock/skills): Identify architecture improvements for testability and AI navigation.
- [prd-to-issues](https://github.com/mattpocock/skills): Break a PRD into independent implementation issues.
- [tdd](https://github.com/mattpocock/skills): Apply red-green-refactor workflows.
- [write-a-prd](https://github.com/mattpocock/skills): Build a PRD from interviews and codebase exploration.

## MCP

- `@sveltejs/mcp`
