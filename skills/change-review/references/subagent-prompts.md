# Subagent Prompts

Give each subagent the target, diff context, expected behavior, and one perspective.

## Base prompt

`Review the diff and relevant code from the [PERSPECTIVE] perspective. Report only actionable, evidence-backed findings with severity, project-relative location, impact, and a fix when clear. Put missing-context concerns in Open Questions. Exclude cosmetic comments and merge decisions.`

## Perspectives

- **Code/QA:** correctness, edge cases, maintainability, tests, regressions.
- **Design:** Is a smaller existing solution adequate? Flag only concrete simpler paths, speculative abstractions, unneeded configuration/dependencies, or a wrong seam; otherwise use Open Questions.
- **Architecture:** boundaries, contracts, coupling, reliability, scalability.
- **Security:** trust boundaries, auth, data handling, injection, secrets, abuse.
- **UX:** accessibility, interaction, responsiveness, copy, regressions.
- **Performance:** hot paths, queries, allocations, caching, latency, throughput.
- **Data:** schema, migrations, consistency, rollback, data loss.
- **Deployment:** CI/CD, configuration, observability, rollout, rollback.

Add perspectives only when the diff warrants them.
