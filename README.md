# Agents

This repository contains local agent definitions under `agents/`, local skills
under `skills/`, and instructions under `instructions/`.

## Instructions

```bash
# Claude:
ln -sf $PWD/instructions/CLAUDE.md ~/.claude/CLAUDE.md

# VSCode Copilot:
ln -sf $PWD/instructions/copilot-instructions.md ~/.copilot/copilot-instructions.md

# Cursor:
mkdir -p ~/.cursor/rules
ln -sf $PWD/instructions/cursor-instructions.mdc ~/.cursor/rules/agents.mdc
```

## Local Agents

Current local agents:

- `architecture`
- `code-qa-engineer`
- `code-reviewer`
- `devops-engineer`
- `performance-engineer`
- `principal-engineer`
- `researcher-engineer`
- `security-engineer`
- `ux-engineer`

Note: Keep this list in sync with the `agents/` directory or generate it from the filesystem.

Link all local agents into `~/.claude/agents` or `~/.cursor/rules/agents`, `~/.copilot/agents` is ignored since Copilot looks at claude agents and copilot agents:

```bash
mkdir -p ~/.claude/agents
mkdir -p ~/.cursor/rules/agents
for f in agents/*.agent.md; do
  ln -sf "$PWD/$f" "$HOME/.claude/agents/$(basename "$f")"
  ln -sf "$PWD/$f" "$HOME/.cursor/rules/agents/$(basename "$f")"
done
```

### Remove agents

```bash
for f in agents/*.agent.md; do
  rm -f "$HOME/.claude/agents/$(basename "$f")"
  rm -f "$HOME/.cursor/rules/agents/$(basename "$f")"
done
```

### External agents

- `svelte-file-editor`: An agent that can read, write, and edit files in a Svelte 5 codebase.

```bash
for agent in \
  https://raw.githubusercontent.com/sveltejs/ai-tools/refs/heads/main/tools/agents/svelte-file-editor.md
do
  filename=$(basename "$agent")
  curl -L "$agent" -o "$HOME/.claude/agents/$filename"
  curl -L "$agent" -o "$HOME/.cursor/rules/agents/$filename"
done
```

### Remove external agents

```bash
for agent in \
  https://raw.githubusercontent.com/sveltejs/ai-tools/refs/heads/main/tools/agents/svelte-file-editor.md
do  filename=$(basename "$agent")
  rm -f "$HOME/.claude/agents/$filename"
  rm -f "$HOME/.cursor/rules/agents/$filename"
done
```

## Local Skills

Current local skills:

- `change-review`: Provides a structured code review of a proposed change, including feedback on correctness, style, maintainability, and potential impacts.
- `github-actions`: Provides best practices and guidance for GitHub Actions workflow design, review, and troubleshooting. Use when you need correct, secure, and maintainable CI/CD workflow recommendations.
- `openai-reusable-prompts`: Designs and improves prompt best practices and formatting structure for OpenAI use cases. Use when you need clear, consistent prompt templates, stronger instruction writing, and better formatted prompt sections.
- `poc`: Produces a focused proof-of-concept plan, feasibility assessment, and minimal experiment artifacts for a feature, integration, or architectural approach.
- `refactor`: Refactor code to improve its structure, readability, and maintainability without changing its external behavior.
- `review`: Review or analyze something, such as a document, code, or data, to provide feedback, identify issues, or suggest improvements.
- `rewrite`: Performs a structured, opinionated, and safety-conscious full rewrite that intentionally introduces breaking changes.

Install all local skills:

```bash
for d in skills/*/; do
  npx skills add "./${d%/}" --global --symlink -y
done
```

## Optional External Skills

- [caveman](https://github.com/mattpocock/skills): Enable compressed, token-efficient assistant responses.
- [grill-me](https://github.com/mattpocock/skills): Deeply interview the user to stress-test a plan or design.
- [improve-codebase-architecture](https://github.com/mattpocock/skills): Identify architecture improvements for testability and AI navigation.
- [to-issues](https://github.com/mattpocock/skills): Break a PRD into independent implementation issues.
- [to-prd](https://github.com/mattpocock/skills): Break a high-level goal into a structured PRD with features, user stories, and acceptance criteria.
- [tdd](https://github.com/mattpocock/skills): Apply red-green-refactor workflows.
- [ubiquitous-language](https://github.com/mattpocock/skills): Establish a common language across the team to improve communication and understanding.
- [svelte-code-writer](https://github.com/sveltejs/ai-tools/tree/main/tools/skills/svelte-code-writer): A skill that can write Svelte code based on user instructions, including components, stores, and more.
- [svelte-core-bestpractices](https://github.com/sveltejs/ai-tools/tree/main/tools/skills/svelte-core-bestpractices): A skill that provides best practices and guidance for writing Svelte code, including component design, state management, and performance optimization.
- [web-design-guidelines](https://github.com/vercel-labs/agent-skills): Follow best practices for web design and user experience.

```bash
for skill in \
  mattpocock/skills/caveman \
  mattpocock/skills/grill-me \
  mattpocock/skills/improve-codebase-architecture \
  mattpocock/skills/to-issues \
  mattpocock/skills/to-prd \
  mattpocock/skills/tdd \
  mattpocock/skills/ubiquitous-language \
  sveltejs/ai-tools/tools/skills/svelte-code-writer \
  sveltejs/ai-tools/tools/skills/svelte-core-bestpractices \
  vercel-labs/agent-skills/skills/web-design-guidelines
do
  npx skills add "$skill" --global --symlink -y
done
```

## MCP

- [@sveltejs/mcp](https://github.com/sveltejs/ai-tools)
