# Agents

This repository contains local agent definitions under `agents/`, local skills
under `skills/`, and instructions under `instructions/`.

## Instructions

```bash
# Codex
ln -sf $PWD/instructions/AGENTS.md ~/.codex/AGENTS.md

# Claude:
ln -sf $PWD/instructions/AGENTS.md ~/.claude/CLAUDE.md

# VSCode Copilot:
ln -sf $PWD/instructions/AGENTS.md ~/.copilot/copilot-instructions.md

# Zed
ln -sf $PWD/instructions/AGENTS.md ~/.config/zed/AGENTS.md

# Cursor:
mkdir -p ~/.cursor/rules
ln -sf $PWD/instructions/cursor-instructions.mdc ~/.cursor/rules/agents.mdc

# Continue
mkdir -p ~/.continue/rules
ln -sf $PWD/instructions/continue-instructions.mdc ~/.cursor/rules/agents.mdc
```

## Local Skills

Current local skills:

- `change-review`: Provides a structured code review of a proposed change, including feedback on correctness, style, maintainability, and potential impacts.
- `github-actions`: Provides best practices and guidance for GitHub Actions workflow design, review, and troubleshooting. Use when you need correct, secure, and maintainable CI/CD workflow recommendations.
- `goal`: Guides agents to complete work against a measurable metric or testable outcome, repeatedly checking progress until the goal passes or a concrete blocker requires user input.
- `openai-reusable-prompts`: Designs and improves prompt best practices and formatting structure for OpenAI use cases. Use when you need clear, consistent prompt templates, stronger instruction writing, and better formatted prompt sections.
  Install all local skills:

```bash
for d in skills/*/; do
  npx skills add "./${d%/}" --global --symlink -y
done
```

Install a single local skill directly from the Github repo without cloning it first:

```bash
npx skills add nathanfaucett/agents/skills/change-review --global --symlink -y
```

### Clean up local skill links:

```bash
for d in skills/*/; do
  npx skills remove "${d%/}" --global -y
done
```

## Optional External Skills

- [Matt Pocock Skills](https://github.com/mattpocock/skills): A collection of skills for software engineering and productivity
- [svelte-code-writer](https://github.com/sveltejs/ai-tools/tree/main/tools/skills/svelte-code-writer): A skill that can write Svelte code based on user instructions, including components, stores, and more.
- [svelte-core-bestpractices](https://github.com/sveltejs/ai-tools/tree/main/tools/skills/svelte-core-bestpractices): A skill that provides best practices and guidance for writing Svelte code, including component design, state management, and performance optimization.
- [web-design-guidelines](https://github.com/vercel-labs/agent-skills): Follow best practices for web design and user experience.
- [open-websearch](https://github.com/aas-ee/open-websearch): A skill for performing web searches and retrieving relevant information.
- [playwriter](https://playwriter.dev): A skill for generating and editing text content, including articles, stories, and scripts.
- [ponytail](https://github.com/DietrichGebert/ponytail): A skill for writing the simpliest and most effective solutions

```bash
for skill in \
  mattpocock/skills/skills/engineering/improve-codebase-architecture \
  mattpocock/skills/skills/engineering/grill-with-docs \
  mattpocock/skills/skills/productivity/grilling \
  mattpocock/skills/skills/engineering/domain-modeling \
  mattpocock/skills/skills/engineering/tdd \
  mattpocock/skills/skills/productivity/handoff \
  sveltejs/ai-tools/tools/skills/svelte-code-writer \
  sveltejs/ai-tools/tools/skills/svelte-core-bestpractices \
  vercel-labs/agent-skills/skills/web-design-guidelines \
  Aas-ee/open-webSearch/skills/open-websearch \
  https://playwriter.dev \
  DietrichGebert/ponytail/skills/ponytail
do
  npx skills add "$skill" --global --symlink -y
done
```

## MCP

- [@sveltejs/mcp](https://github.com/sveltejs/ai-tools)
- [open-websearch](https://github.com/aas-ee/open-websearch)
