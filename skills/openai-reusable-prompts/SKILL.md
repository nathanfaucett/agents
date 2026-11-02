---
name: openai-reusable-prompts
description: "Designs and improves prompt best practices and formatting structure for OpenAI use cases. Invoke when reusable prompts require clear instruction hierarchy, consistent formatting, and strong output contracts."
argument-hint: "Describe your use case, required inputs, output format, and constraints."
---

# OpenAI Reusable Prompts

## Summary

This skill focuses only on writing high-quality prompts and formatting them for clarity, consistency, and better model steerability.

It helps define prompt structure, wording quality, examples, and formatting patterns for reliable outputs.

See [references](references/) for detailed OpenAI prompt engineering guides.

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

## Reference material

See `references/` for:

- [procedure.md](references/procedure.md) — 8-step prompt authoring procedure
- [branching-logic.md](references/branching-logic.md) — Conditional prompt patterns
- [quality-criteria.md](references/quality-criteria.md) — Completion criteria
- [api-deployment-guidance.md](references/api-deployment-guidance.md) — OpenAI API deployment notes
- [agent-skill-packaging.md](references/agent-skill-packaging.md) — Skill packaging guidance
- [reproducibility-versioning.md](references/reproducibility-versioning.md) — Versioning and reproducibility
- [deliverables.md](references/deliverables.md) — Expected outputs
- [code-managed-prompt-template.md](references/code-managed-prompt-template.md) — Code-managed prompt template
- [skill-authoring-addon-template.md](references/skill-authoring-addon-template.md) — Skill authoring add-on template
- [prompt-formatting-checklist.md](references/prompt-formatting-checklist.md) — Prompt formatting checklist
- [prompt-starter-patterns.md](references/prompt-starter-patterns.md) — Prompt starter patterns
- [guardrails.md](references/guardrails.md) — Hard constraints
