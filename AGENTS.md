# Agents

## Rules

### Agents Location

- All agents go under `agents/<agent-name>.agent.md`. Use `ln -sf agents/<agent-name>.agent.md ~/.claude/agents/<agent-name>.agent.md` to link the files to global agents folder.

### <agent-name>.agent.md Frontmatter Constraints
- `name` is **required**. Max 64 chars. Lowercase letters, numbers, hyphens only. Cannot start/end with hyphen. No consecutive hyphens. **Must match file name exactly.**
- `name` **cannot contain** "anthropic" or "claude".
- `description` is **required**. Max 1024 chars. Write in **third person**. Must describe both what it does AND when to trigger it (Claude uses this field to decide when to invoke).

### Skill Location
- All skills go under `skills/<skill-name>/SKILL.md`. The `npx skills` CLI only discovers skills in the `skills/` directory at repo root.

Follows the [Agent Skills spec](https://agentskills.io/specification) for skills.sh compatibility.

### SKILL.md Frontmatter Constraints
- `name` is **required**. Max 64 chars. Lowercase letters, numbers, hyphens only. Cannot start/end with hyphen. No consecutive hyphens. **Must match parent directory name exactly.**
- `name` **cannot contain** "anthropic" or "claude".
- `description` is **required**. Max 1024 chars. Write in **third person**. Must describe both what it does AND when to trigger it (Claude uses this field to decide when to invoke).

### Skill Body
- Keep under 500 lines. Move detailed reference material to `references/` subdirectory within the skill folder.
- Helper scripts go in `scripts/` subdirectory within the skill folder.

### Installation
- ```bash
  npx skills add <skill-name>
  ```