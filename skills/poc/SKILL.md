---
name: poc
description: |
   Produces a focused proof of concept plan, feasibility assessment, and minimal experiment artifacts for a feature, integration, or architectural approach. Invoke when a team needs to validate an idea quickly, compare implementation options, or de-risk a technical direction before committing to production work.
---

## Summary
Use this skill to turn a rough idea into a fast, testable proof of concept. The goal is to answer questions like "can this work?", "what is the cheapest experiment to prove it?", and "what evidence do we need before building the real version?"

This skill is optimized for speed and clarity, not production readiness. Outputs may include lightweight code sketches, experiment plans, test harnesses, rough architecture notes, and a go-or-no-go recommendation.

The primary execution agent for this skill is `researcher-engineer.agent.md`. Use that agent for the actual feasibility analysis, trade-off evaluation, and POC design work.

## When to use
- A feature idea is still vague and needs a concrete experiment.
- A team wants to compare two or more implementation approaches before committing.
- You need a lightweight prototype, benchmark plan, or integration spike.
- You need a recommendation based on constraints such as time, cost, platform, scale, or risk.
- You want a small artifact that helps engineering decide whether to proceed, iterate, or stop.

## When not to use
- The user needs production-ready code, full hardening, or long-term maintainability work.
- The problem is already well understood and only needs routine implementation.
- The task requires large-scale refactoring across the repository rather than a narrow experiment.

## Inputs
Provide as many of these as possible:

- Problem statement or idea to validate.
- Constraints such as deadline, staffing, platform, budget, dependencies, or compliance requirements.
- Existing code, APIs, schemas, architecture notes, or competing approaches.
- Success criteria for the proof of concept.
- Any explicit limits on touching files, installing dependencies, or running commands.

If the request is underspecified, ask only for the minimum missing information needed to define a meaningful experiment.

## Outputs
Produce a compact POC package containing:

1. Feasibility summary
   - Clear recommendation: `Go`, `Iterate`, or `Stop`
   - Main reasons and key unknowns
2. Proposed experiment
   - Smallest viable scope that can answer the core question
   - Steps to execute the experiment
3. Success criteria
   - Observable outcomes, benchmarks, or pass/fail thresholds
4. Risks and trade-offs
   - Technical risks, assumptions, and likely failure modes
5. Artifacts
   - Minimal code sketch, sample config, benchmark plan, or implementation notes when useful
6. Next steps
   - What to do if the experiment succeeds, fails, or is inconclusive

## Required operating mode
- Delegate the core investigation to `researcher-engineer.agent.md`.
- Frame the work as a focused research-and-prototype task, not a production implementation.
- Prefer the smallest experiment that produces credible evidence.
- Keep artifacts intentionally narrow, easy to inspect, and safe to discard.
- Explicitly note where the POC cuts corners compared with production design.

## Step-by-step process
1. Clarify the question
   - Identify the exact uncertainty to resolve.
   - Convert broad asks into a concrete validation target.
2. Define constraints
   - Timebox, environment limits, acceptable dependencies, data sensitivity, and performance targets.
3. Choose the experiment shape
   - Pick one of: code spike, benchmark, integration mock, architecture comparison, or test harness.
4. Invoke `researcher-engineer.agent.md`
   - Ask it to evaluate feasibility, compare approaches if needed, and design the minimal POC.
   - If useful, have it produce tiny prototype snippets or a benchmark outline.
5. Review the recommendation
   - Ensure the result answers the original question with evidence, not just opinion.
6. Package the result
   - Summarize feasibility, proposed steps, success criteria, and next actions.
7. Only then expand scope
   - If the user wants implementation after the POC, treat that as a separate phase.

## Branching logic
- If the user is deciding between approaches:
   - Ask `researcher-engineer` for a comparison with trade-offs, estimated effort, and a recommended experiment.
- If the user wants to know whether something can scale:
  - Ask for a benchmark-oriented POC with measurable thresholds.
- If the user wants an integration spike:
  - Ask for a mock or minimal end-to-end path proving the critical interface.
- If the user asks for code immediately:
  - Still frame it as a limited POC and document the shortcuts.
- If the user actually needs production work:
  - State that the POC only de-risks the idea and should not be treated as final implementation.

## Quality bar
The skill is successful when it:

- Resolves a concrete uncertainty.
- Produces a recommendation that is evidence-oriented.
- Defines explicit acceptance criteria for the experiment.
- Minimizes time and scope while preserving decision value.
- Makes non-production assumptions visible.

## Guardrails
- Do not present POC code as production-ready.
- Do not request or store secrets.
- Avoid broad repository changes unless the user explicitly asks for them.
- Prefer mocks, fixtures, and public sample data over sensitive data.
- Call out missing validation work, hardening, tests, and operational concerns.

## Prompt patterns for `researcher-engineer`
- "Design a 1-2 day POC to validate whether {approach} can satisfy {constraint}. Include steps, minimal artifacts, and success criteria."
- "Compare approaches A, B, and C for {problem}. Recommend the smallest experiment that would let us choose confidently."
- "Produce a lightweight benchmark plan to test whether {system} can handle {load target}. Include pass/fail thresholds."
- "Sketch a minimal integration spike for {service or API}. Show what we need to mock, what to measure, and what would make the spike successful."

## Example requests
- "Validate whether we can stream updates from provider X into our app with less than 500ms end-to-end latency."
- "Compare a queue-based design and a direct RPC design for this workflow and propose a POC."
- "Create a tiny prototype plan to test whether local-first sync is viable for our offline mode."
- "Research whether we can replace vendor Y with an open-source alternative and define a proof-of-concept."