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

4. Apply operational best practices.
- Keep skills discoverable. In `SKILL.md`, include: Use when, Don't use when, how to run, expected outputs, gotchas, and key edge cases.
- Include both positive and negative routing examples so invocation is predictable.
- Iterate on `name`, `description`, and routing examples before changing implementation when routing is inconsistent.
- Prefer zip uploads for reliability and reproducibility in API workflows.
- Pin skill versions in production. Use explicit versions for production paths, and only float to `"latest"` in non-production workflows.
- Pin model and skill version together in production workflows when reproducibility matters.
- Design skill scripts like tiny CLIs: runnable from command line, deterministic stdout, clear usage/errors, and known output paths.
- Use `${SKILL_DIR}` for executable paths referenced from skill docs. `SKILL_DIR` is the absolute directory containing the active `SKILL.md`.
- Apply `${SKILL_DIR}` to `scripts/...` command examples and other executable file paths. Prose-only mentions of `references/...` and `templates/...` can stay relative.
- Do not traverse to sibling skills with paths like `../other-skill/scripts/...`; invoke that skill directly and use its own documented `${SKILL_DIR}` examples.
- Avoid duplicating full procedures in system prompts; keep global behavior in system prompts and reusable procedures in skills.
- Treat network-enabled skills as high-risk: use strict allowlists, document allowed data egress, and treat tool output as untrusted.
- Choose models that reliably execute multi-step workflows for skill-heavy tasks; add explicit verification and output checks in `SKILL.md` when flows are brittle.
- Respect platform limits for API-hosted skills: exactly one manifest (`SKILL.md` or `skill.md`), max zip size 50 MB, max files per skill version 500, and max uncompressed file size 25 MB.

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
- Skill routing guidance includes both "Use when" and "Don't use when" expectations.
- Production reproducibility guidance covers skill version pinning and model-version pairing.
- Security guidance covers network allowlists and explicit data-egress boundaries for network-enabled skills.
- Reliability guidance covers CLI-style execution and explicit verification/output checks.
- Operational limits for API-hosted skills are explicit.
- Installation command is present and correct.

## Guardrails
- Do not place agents outside `agents/`.
- Do not place skills outside `skills/<skill-name>/SKILL.md`.
- Do not use names that violate character or length constraints.
- Do not omit trigger guidance from `description` fields.