# Subagent Prompts

Give each subagent the target, diff context, expected behavior, and one perspective.

## Base prompt

`Think about this code from the [PERSPECTIVE] perspective. Inspect the diff and relevant surrounding code. Report only actionable, evidence-backed findings. For each finding include severity, project-relative file, line range, impact, and a fix when clear. Put missing-context concerns under Open Questions. Do not include cosmetic comments or decide whether the change should merge.`

## Perspectives

- **Code/QA:** correctness, edge cases, maintainability, tests, regressions.
- **Architecture:** boundaries, contracts, coupling, reliability, scalability.
- **Security:** trust boundaries, auth, data handling, injection, secrets, abuse cases.
- **UX:** accessibility, interaction, responsive behavior, copy, user regressions.
- **Performance:** hot paths, queries, allocations, caching, latency, throughput.
- **Data:** schema changes, migrations, consistency, rollback, data loss.
- **Deployment:** CI/CD, configuration, observability, rollout, rollback.

Ask for another perspective only when the diff makes it relevant.