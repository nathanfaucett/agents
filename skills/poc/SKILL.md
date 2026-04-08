---
name: poc
description: |
   Produces a focused proof of concept plan, feasibility check, and minimal experiment for a feature, integration, or approach. Invoke when quickly validating ideas, comparing options, or de-risking before committing to full implementation.
---

## Summary
Quickly turn an idea into a testable proof of concept. Answer: Can this work? What is the smallest experiment to prove it? What evidence do we need before building the real thing?

Outputs: code sketch, experiment plan, test harness, architecture notes, and a go/iterate/stop recommendation.

Delegate feasibility analysis and POC design to a research-focused engineering agent.

## When to use
- Idea is vague and needs validation.
- Comparing multiple approaches.
- Need a quick prototype, benchmark, or integration spike.
- Want a recommendation based on constraints (time, cost, platform, risk).

## When not to use
- User needs production-ready code or full hardening.
- Problem is routine and well understood.
- Task requires broad refactoring, not a narrow experiment.

## Inputs
Provide:
- Problem statement or idea
- Constraints (deadline, platform, budget, dependencies, compliance)
- Existing code, APIs, or competing approaches
- Success criteria
- Any limits on files, dependencies, or commands

If unclear, ask only for the minimum info needed to define an experiment.

## Outputs
Produce a compact POC package:
1. Feasibility summary: Go / Iterate / Stop, with reasons
2. Proposed experiment: minimal steps to answer the core question
3. Success criteria: observable outcomes or pass/fail
4. Risks/trade-offs: main risks, assumptions, failure modes
5. Artifacts: code sketch, config, or notes if useful
6. Next steps: what to do if experiment succeeds, fails, or is inconclusive

## Operating mode
- Always delegate core investigation to a research-focused engineering agent.
- Frame as research/prototype, not production.
- Prefer the smallest credible experiment.
- Keep artifacts minimal and easy to discard.
- Note where POC cuts corners vs. production.

## Process
1. Clarify the core question/uncertainty.
2. Define constraints (time, env, dependencies, data, perf).
3. Pick experiment type: code spike, benchmark, integration mock, comparison, or test harness.
4. Invoke a research-focused engineering agent for feasibility, comparison, and minimal POC design.
5. Review: ensure evidence, not just opinion.
6. Package: summarize feasibility, steps, criteria, next actions.
7. Expand scope only if user requests implementation after POC.

## Branching logic
- Comparing approaches? Ask for comparison, trade-offs, effort, and experiment.
- Testing scale? Ask for benchmark POC with thresholds.
- Integration spike? Ask for minimal mock or end-to-end path.
- User wants code? Frame as limited POC, document shortcuts.
- User needs production? State POC is not production, only de-risks idea.

## Quality bar
Success =
- Concrete uncertainty resolved
- Evidence-based recommendation
- Explicit acceptance criteria
- Minimal time/scope, high decision value
- Non-production shortcuts are visible

## Guardrails
- Never present POC code as production-ready.
- Never request/store secrets.
- Avoid broad repo changes unless user asks.
- Prefer mocks/sample data over sensitive data.
- Call out missing validation, hardening, tests, ops.

## Prompt patterns for research-focused engineering agents
- "Design a 1-2 day POC to validate if {approach} meets {constraint}. Include steps, minimal artifacts, and success criteria."
- "Compare approaches A, B, C for {problem}. Recommend the smallest experiment to choose confidently."
- "Make a lightweight benchmark plan to test if {system} handles {load target}. Include pass/fail."
- "Sketch a minimal integration spike for {service/API}. Show what to mock, measure, and what makes it successful."

## Example requests
- "Validate streaming updates from provider X with <500ms latency."
- "Compare queue vs RPC for workflow and propose POC."
- "Prototype plan to test local-first sync for offline mode."
- "Research replacing vendor Y with open-source and define POC."