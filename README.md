# Agents

```bash
npx skills add $skill
```

## Skills

- agent-ready: Analyze codebase for gaps that hinder autonomous agents; give prioritized recommendations.
- subagents-tasks: Create an iterative plan file with Pending/In progress/Done and next-subagent signals.
- subagents-tasks-run: Execute the next pending subagent task and update the plan file with progress.

### External

- sveltejs/ai-tools/plugins/cursor/svelte/skills/svelte-code-writer: CLI tools for Svelte 5 documentation lookup and code analysis
- sveltejs/ai-tools/plugins/cursor/svelte/skills/svelte-core-bestpractices: Guidance on writing fast, robust, modern Svelte code
- mattpocock/skills/grill-me: Interview the user deeply to stress-test a plan or design.
- mattpocock/skills/improve-codebase-architecture: Find architectural improvements to make code more testable and AI-navigable.
- mattpocock/skills/prd-to-issues: Break a PRD into independent GitHub issues (vertical slices).
- mattpocock/skills/tdd: Test-driven development workflow (red-green-refactor).
- mattpocock/skills/write-a-prd: Create a PRD via interview and code exploration, and submit as a GitHub issue.

## MCP

- @sveltejs/mcp