---
name: obsidian-cli
description: "This skill provides command-line interaction with Obsidian for vault management (reading, creating, searching notes, tasks, properties) and development support (reloading plugins, debugging via DOM inspection/scripting). Use when performing automation or development tasks using the CLI."
---

# Obsidian CLI

Use the `obsidian` CLI to interact with a running Obsidian instance. Requires Obsidian to be open. Full documentation: https://help.obsidian.md/cli

## When to use

- Automation of note creation and modification (e.g., daily notes).
- Bulk data extraction or search across the vault.
- Integrating Obsidian with external workflows (e.g., generating tasks for a project management tool).
- Development support (reloading plugins, debugging via CLI).

## When not to use

- When Obsidian is not running or accessible.
- For complex UI interactions that require the graphical interface (e.g., setting themes, managing attachments graphically).
- When simple file system operations suffice (use standard OS commands instead).

## Syntax & Usage

**Parameters** take a value with `=`. Quote values with spaces:

```bash
obsidian create name="My Note" content="Hello world"
```

Run `obsidian help` for all available commands.

## Worked Example (Note Creation)

**Input:** Need to create a new note named "Meeting Summary" with content "Discussed project goals."
**Command:** `obsidian create name="Meeting Summary" content="Discussed project goals."`
**Expected Output:** Confirmation that the note was created at the specified path within the vault.

## Gotchas / Guardrails

- Obsidian must be running for the CLI to function.
- Paths are relative to the active Obsidian vault.
- Ensure you use quotes when values contain spaces.

## Procedure

1. **Identify Goal:** Determine the specific vault operation (create, read, search, tasks, properties).
2. **Check Environment:** Verify Obsidian is running.
3. **Construct Command:** Use `obsidian <command> name="<value>" content="<value>"`.
4. **Execute and Verify:** Run the command and check the terminal output for successful execution confirmation.
