# Agents

## Agents

```bash
mkdir -p ~/.claude/agents
ln -sf ./agents/implementation-researcher.agent.md ~/.claude/agents/principal-engineer.agent.md
ln -sf ./agents/principal-engineer.agent.md ~/.claude/agents/principal-engineer.agent.md
ln -sf ./agents/security-engineer.agent.md ~/.claude/agents/security-engineer.agent.md
ln -sf ./agents/ux-ui-engineer.md.agent.md ~/.claude/agents/ux-ui-engineer.md.agent.md
```

## Skills

- agent-ready: Analyze codebase for gaps that hinder autonomous agents; give prioritized recommendations.
- subagents-tasks: Create an iterative plan file with Pending/In progress/Done and next-subagent signals.
- subagents-tasks-run: Execute the next pending subagent task and update the plan file with progress.

### 

```bash
npx skills add \
  ./skills/agent-ready \
  ./skills/subagents-tasks \
  ./skills/subagents-tasks-run
```
### External Skills Used

```bash
npx skills add \
	mattpocock/skills/grill-me \
    mattpocock/skills/improve-codebase-architecture \
    mattpocock/skills/prd-to-issues \
    mattpocock/skills/tdd \
    mattpocock/skills/write-a-prd
```

- [grill-me](https://github.com/mattpocock/skills): Interview the user deeply to stress-test a plan or design.
- [improve-codebase-architecture](https://github.com/mattpocock/skills): Find architectural improvements to make code more testable and AI-navigable.
- [prd-to-issues](https://github.com/mattpocock/skills): Break a PRD into independent GitHub issues (vertical slices).
- [tdd](https://github.com/mattpocock/skills): Test-driven development workflow (red-green-refactor).
- [write-a-prd](https://github.com/mattpocock/skills): Create a PRD via interview and code exploration, and submit as a GitHub issue.

## MCP

- @sveltejs/mcp