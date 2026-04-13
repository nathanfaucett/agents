# Agents

This repository contains local agent definitions under `agents/`, local skills
under `skills/`, and instructions under `instructions/`.

## Instructions

```bash
# VSCode Copilot instructions:
ln -sf $PWD/instructions/CLAUDE.md ~/.claude/CLAUDE.md
ln -sf $PWD/instructions/copilot-instructions.md ~/.copilot/copilot-instructions.md
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

Link all local agents into `~/.claude/agents`:

```bash
mkdir -p ~/.claude/agents
for f in agents/*.agent.md; do
  ln -sf "$PWD/$f" "$HOME/.claude/agents/$(basename "$f")"
done
```

## Local Skills

Current local skills:

- `agent-ready`: Analyzes a codebase to identify gaps, inconsistencies, and friction points that hinder autonomous agents or developers from effectively understanding, modifying, and extending the project.
- `change-review`: Provides a structured code review of a proposed change, including feedback on correctness, style, maintainability, and potential impacts.
- `github-actions`: Provides best practices and guidance for GitHub Actions workflow design, review, and troubleshooting. Use when you need correct, secure, and maintainable CI/CD workflow recommendations.
- `openai-reusable-prompts`: Designs and improves prompt best practices and formatting structure for OpenAI use cases. Use when you need clear, consistent prompt templates, stronger instruction writing, and better formatted prompt sections.
- `poc`: Produces a focused proof-of-concept plan, feasibility assessment, and minimal experiment artifacts for a feature, integration, or architectural approach.
- `refactor`: Refactor code to improve its structure, readability, and maintainability without changing its external behavior.
- `review`: Review or analyze something, such as a document, code, or data, to provide feedback, identify issues, or suggest improvements.
- `rewrite`: Performs a structured, opinionated, and safety-conscious full rewrite that intentionally introduces breaking changes.
- `specs`: Runs a deterministic Spec-Driven Development (SDD) workflow using fixed files, hard phase gates, and explicit redirection.

Install all local skills:

```bash
for d in skills/*/; do
  npx skills add "./${d%/}" --global --symlink -y
done
```

## Optional External Skills

```bash
for skill in \
  mattpocock/skills/grill-me \
  mattpocock/skills/improve-codebase-architecture \
  mattpocock/skills/prd-to-issues \
  mattpocock/skills/tdd \
  mattpocock/skills/write-a-prd \
  JuliusBrussee/caveman
do
  npx skills add "$skill" --global --symlink -y
done
```

- [grill-me](https://github.com/mattpocock/skills): Deeply interview the user to stress-test a plan or design.
- [improve-codebase-architecture](https://github.com/mattpocock/skills): Identify architecture improvements for testability and AI navigation.
- [prd-to-issues](https://github.com/mattpocock/skills): Break a PRD into independent implementation issues.
- [tdd](https://github.com/mattpocock/skills): Apply red-green-refactor workflows.
- [write-a-prd](https://github.com/mattpocock/skills): Build a PRD from interviews and codebase exploration.
- [caveman](https://github.com/JuliusBrussee/caveman): Enable compressed, token-efficient assistant responses.

## MCP

- `@sveltejs/mcp`

## CLI Tools

- [rtk](https://github.com/rtk-ai/rtk?tab=readme-ov-file#supported-ai-tools)