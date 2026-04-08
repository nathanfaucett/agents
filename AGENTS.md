# Agents

## Summary
This document defines the repository rules for agent files and skill files.

It standardizes file locations, frontmatter constraints, and size/structure limits so agents and skills remain compatible with existing tooling.

## When to use
- You are creating a new agent definition.
- You are creating a new skill package.
- You are reviewing or fixing frontmatter for agent/skill metadata.
- You are validating repository compatibility with `npx skills`.

## When not to use
- You need runtime implementation guidance unrelated to agent/skill metadata.
- You are making feature-specific code decisions outside file-format constraints.

## Procedure
### Agent flow
1. Place agent files in the correct location.
- Store all agents under `agents/<agent-name>.agent.md`.
- Link agent files to the global agents folder with:

```bash
ln -sf agents/<agent-name>.agent.md ~/.claude/agents/<agent-name>.agent.md
```

2. Validate `<agent-name>.agent.md` frontmatter.
- `name` is required.
- `name` max length is 64 characters.
- `name` must use lowercase letters, numbers, and hyphens only.
- `name` cannot start or end with a hyphen.
- `name` cannot contain consecutive hyphens.
- `name` must match the file name exactly.
- `name` cannot contain `anthropic` or `claude`.
- `description` is required.
- `description` max length is 1024 characters.
- `description` must be written in third person.
- `description` must describe both what it does and when to trigger it.

### Skill flow
1. Place skill files in the correct location.
- Store all skills under `skills/<skill-name>/SKILL.md`.
- The `npx skills` CLI only discovers skills in the `skills/` directory at repository root.
- Follow the [Agent Skills spec](https://agentskills.io/specification) for skills.sh compatibility.

2. Validate `SKILL.md` frontmatter.
- `name` is required.
- `name` max length is 64 characters.
- `name` must use lowercase letters, numbers, and hyphens only.
- `name` cannot start or end with a hyphen.
- `name` cannot contain consecutive hyphens.
- `name` must match the parent directory name exactly.
- `name` cannot contain `anthropic` or `claude`.
- `description` is required.
- `description` max length is 1024 characters.
- `description` must be written in third person.
- `description` must describe both what it does and when to trigger it.

3. Validate skill body structure.
- Keep `SKILL.md` under 500 lines.
- Move detailed reference material into `references/` within the skill folder.
- Put helper scripts in `scripts/` within the skill folder.

## Installation
Use the following command to install a skill:

```bash
npx skills add <skill-name>
```

## Quality criteria
A rules update is complete when all are true:
- Agent and skill file locations are explicit.
- Frontmatter constraints are explicit for both agents and skills.
- Naming restrictions include all forbidden patterns.
- Skill body limits and folder conventions are explicit.
- Installation command is present and correct.

## Guardrails
- Do not place agents outside `agents/`.
- Do not place skills outside `skills/<skill-name>/SKILL.md`.
- Do not use names that violate character or length constraints.
- Do not omit trigger guidance from `description` fields.