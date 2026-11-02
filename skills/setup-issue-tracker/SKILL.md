---
name: setup-issue-tracker
description: "Configure this repo for the engineering skills: set up its issue tracker, triage label vocabulary, and domain doc layout. Run once before first use of the other engineering skills."
disable-model-invocation: true
---

# Setup Issue Tracker

Scaffold the per-repo configuration that the engineering skills assume:

- **Issue tracker**: where issues live (GitHub by default; local markdown is also supported out of the box)
- **Triage labels**: the strings used for the five canonical triage roles
- **Domain docs**: where `GLOSSARY.md` live, and the consumer rules for reading them

This is a prompt-driven skill, not a deterministic script. Explore, present what you found, confirm with the user, then write.

## Process

See [references/process.md](references/process.md) for the full 5-step process:

1. **Explore** — Detect repo state (GitHub/GitLab/local, triage skill, monorepo)
2. **Present findings and ask** — Section A (tracker), Section B (labels), Section C (domain docs)
3. **Confirm and edit** — Show drafts for AGENTS.md/CLAUDE.md and docs/agents/*.md
4. **Write** — Update AGENTS.md/CLAUDE.md and write docs files from seed templates in `references/`
5. **Done** — Confirm completion
