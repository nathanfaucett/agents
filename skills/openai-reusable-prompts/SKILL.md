---
name: openai-reusable-prompts
description: 'Designs and improves prompt best practices and formatting structure for OpenAI use cases. Use when you need clear, consistent prompt templates, stronger instruction writing, and better formatted prompt sections.'
argument-hint: 'Describe your use case, required inputs, output format, and constraints.'
---

# OpenAI Reusable Prompts

## Summary
This skill focuses only on writing high-quality prompts and formatting them for clarity, consistency, and better model steerability.

It helps define prompt structure, wording quality, examples, and formatting patterns for reliable outputs.

## When to use
- You want better-written prompts with clear instruction hierarchy.
- You need consistent prompt formatting across tasks or teams.
- You want reusable prompt templates with strong examples and constraints.
- You need to improve ambiguous prompts that produce inconsistent output.
- You are standardizing a stable, repeatable procedure that multiple agents or teams should share.
- You want reusable behavior that can be versioned and improved independently over time.

## When not to use
- You are asking for API integration code, deployment, or version rollout strategy.
- You need runtime implementation details instead of prompt writing guidance.
- The request is truly one-off and does not need reusable structure.
- The procedure changes daily and has not stabilized yet.
- The task primarily requires live external state or side effects; use tools or APIs for that.

## Required inputs
Provide or infer:
- Outcome: what the prompt must produce.
- Audience or task type: who uses the output and for what.
- Output contract: free text, markdown, JSON, or schema-constrained output.
- Constraints: style, tone, safety limits, and forbidden behavior.

If relevant, also capture example inputs and expected outputs.

If details are missing, ask only the minimum needed to author a usable v1 template.

## Routing guidance and edge cases
- Prefer triggering this skill for reusable procedures, stable writing standards, or organization-wide prompt conventions.
- Avoid triggering this skill when the user asks for ad hoc brainstorming that does not require strict structure.
- Include both positive and negative examples in your authored prompt package to improve routing accuracy.
- If routing is inconsistent, iterate on frontmatter `name`, `description`, and examples before changing scripts or runtime code.

## Procedure
1. Define the target behavior.
- Write a one-sentence objective for the prompt.
- Define success as observable output criteria.

2. Set instruction hierarchy.
- Put high-priority behavior first.
- Separate required rules from preferences.
- State what the model must do and must not do.

3. Draft the prompt with consistent formatting.
- Use clear section headers in this order when useful: `Identity`, `Instructions`, `Examples`, `Context`.
- Keep instructions explicit and testable.
- Use placeholders for dynamic values, for example `{{customer_name}}`.

4. Strengthen formatting discipline.
- Use bullet lists for constraints and output rules.
- Use XML-style delimiters for example I/O blocks when helpful.
- Avoid long paragraphs that hide requirements.

5. Add examples that teach behavior.
- Include at least 2 examples when output format is strict.
- Cover common and edge inputs.
- Keep examples consistent with stated rules.

6. Add output contract checks.
- Specify exact format expectations (fields, ordering, allowed labels).
- Add explicit negative constraints to reduce drift.

7. Revise for clarity and brevity.
- Remove redundant instructions.
- Replace vague wording with measurable requirements.
- Keep only context that changes model decisions.

8. Add operational readiness notes when the prompt will be used as a skill artifact.
- Include when to use, how to run, expected outputs, and gotchas.
- Add explicit verification steps and output checks for brittle multi-step workflows.

## Branching logic
- If output must follow strict schema:
Use explicit field-level formatting rules and schema-like examples.

- If behavior differs by customer or locale:
Keep global rules stable and isolate locale-specific rules in dedicated context blocks.

- If the prompt is long and complex:
Split sections clearly and remove non-essential context.

- If task is reasoning-heavy and under-specified:
Start with goal + constraints, then tighten with examples after observing failures.

- If task is deterministic and format-sensitive:
Use strict format instructions and highly specific examples.

## Quality criteria
A skill run is complete when all are true:
- Prompt objective and output contract are explicit.
- Instruction hierarchy is clear (must vs should).
- Formatting is consistent across sections.
- Examples reinforce the required behavior and format.
- Ambiguous wording has been removed.
- Negative trigger examples are present for routing precision.
- Verification steps and output checks are defined when workflow execution is multi-step.

## Operational best practices (skills in API)
Apply these when the prompt template is packaged in a skill context.

1. Keep skills discoverable.
- Use clear frontmatter `name` and `description`.
- Keep explicit "Use when" and "Do not use when" guidance.
- Include negative examples with positive examples.

2. Keep system prompts lean.
- Put stable, reusable procedures in skills.
- Keep global policies and always-on behavior in system prompts.
- Do not duplicate full skill procedures in system prompts.

3. Design script-backed skills like tiny CLIs.
- Ensure commands run from the command line.
- Require deterministic stdout for key status and results.
- Fail loudly with clear usage or validation errors.
- Write outputs to known, documented file paths.

4. Include worked examples in skill assets.
- Provide inputs, commands, and expected outputs.
- Cover normal cases and at least one edge case.

5. Add explicit verification gates.
- Validate output format and required fields.
- Confirm artifacts exist at expected paths.
- Add checks that catch partial completion.

6. Be cautious with network access.
- Prefer no-network execution when possible.
- If network access is required, use strict allowlists and explicit data-egress constraints.

7. Use a model that can reliably complete multi-step workflows.
- If execution is brittle, simplify the workflow and strengthen verification guidance.

## Reproducibility and versioning
- Prefer zip bundles for portability, reliability, and easier version management.
- Pin explicit skill versions in production when reproducibility matters.
- Use floating `latest` only when intentional and monitored.
- If version is omitted, behavior follows the platform default version pointer.
- Pin model version and skill version together for stable cross-deployment behavior.

## Deliverables
Produce:
1. Reusable prompt template text (dashboard-ready)
2. Prompt formatting checklist
3. Improved example set (good and edge cases)
4. Prompt rewrite notes explaining key clarity changes
5. Operational readiness notes (how to run, expected outputs, gotchas, verification checks)

## Dashboard authoring template
Use this as a starting point for the reusable prompt body in the OpenAI Dashboard.

```markdown
# Identity
You are an assistant that {role-purpose}.

# Instructions
- Primary objective: {objective}
- Required constraints: {constraints}
- Output requirements: {format-rules}
- Never do: {prohibited-behaviors}

# Examples
<input id="example-1">
{example-input-1}
</input>
<output id="example-1">
{example-output-1}
</output>

# Context
{stable-reference-context}
```

## Skill authoring add-on template (for runnable skill workflows)
Use this companion template when the prompt package will be executed as a skill with scripts or assets.

```markdown
# When to use
- {positive-trigger-1}
- {positive-trigger-2}

# Do not use when
- {negative-trigger-1}
- {negative-trigger-2}

# Inputs
- {input-contract}

# How to run
- {install-command}
- {run-command}

# Expected outputs
- {path-1}: {artifact-description}
- {path-2}: {artifact-description}

# Gotchas
- {failure-mode-1}
- {failure-mode-2}

# Verification checks
- {check-1}
- {check-2}

# Worked examples
<input id="example-1">
{example-input-1}
</input>
<run id="example-1">
{example-command-1}
</run>
<output id="example-1">
{example-output-1}
</output>
```

## Prompt formatting checklist
Use this checklist for every prompt revision.

```markdown
- [ ] Objective is one sentence and testable
- [ ] Required behavior is listed before optional preferences
- [ ] Output format is explicit and easy to verify
- [ ] At least 2 examples are present for strict formats
- [ ] Negative constraints are included
- [ ] Context is minimal and relevant
- [ ] No contradictory instructions
- [ ] "Use when" and "Do not use when" routing guidance is explicit
- [ ] At least one negative trigger example is included
- [ ] Reproducibility notes are present (versioning and packaging intent)
- [ ] Verification steps and output checks are defined for multi-step workflows
```

## Prompt starter patterns
- "Rewrite this prompt using Identity, Instructions, Examples, Context sections while preserving intent: {prompt}."
- "Tighten this prompt to eliminate ambiguity and enforce this format: {format-rules}."
- "Add high-quality examples to teach this behavior: {objective}."
- "Convert this long paragraph prompt into a concise, structured prompt with explicit constraints."

## Guardrails
- Do not include secrets or private data in prompt text.
- Do not mix conflicting instructions in multiple sections.
- Do not leave output format implied when precision is required.
- Do not overfit to one example; use diverse examples.
- Do not duplicate full skill procedures inside system prompts.
- Do not rely on open network access without strict allowlists and explicit data-egress rules.
- Do not leave output paths undefined for script-backed workflows.
