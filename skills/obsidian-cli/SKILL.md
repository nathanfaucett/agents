---
name: obsidian-cli
description: This skill provides command-line interaction with Obsidian for vault management (reading, creating, searching notes, tasks, properties) and development support (reloading plugins, debugging via DOM inspection/scripting). Use when performing automation or development tasks using the CLI.
---

# Obsidian CLI

Use the `obsidian` CLI to interact with a running Obsidian instance. Requires Obsidian to be open.

## Command reference

Run `obsidian help` to see all available commands. This is always up to date. Full docs: https://help.obsidian.md/cli

## Syntax

**Parameters** take a value with `=`. Quote values with spaces:

```bash
obsidian create name="My Note" content="Hello world"
```
