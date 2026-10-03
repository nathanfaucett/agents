---
title: Prompt engineering
shortTitle: Prompt engineering
intro: Strategies for writing effective prompts with the OpenAI Responses API and Chat Completions API.
---

## Choosing a model

- **Reasoning models** generate an internal chain of thought; excel at complex tasks and multi-step planning. Slower and more expensive.
- **GPT models** are fast, cost-efficient, and highly intelligent; benefit from explicit instructions.
- **Large and small models** trade off speed, cost, and intelligence.

When in doubt, `gpt-6-astra` offers a strong default for general-purpose text generation.

## Message roles and instruction following

Use `instructions` (Responses API) or `developer` messages (Chat Completions) for high-level behavior. They take priority over `user` input.

| Role | Priority | Purpose |
| --- | --- | --- |
| `developer` | Highest | Application rules and business logic |
| `user` | Lower | Inputs and configuration |
| `assistant` | Lowest | Model-generated responses |

**Important**: `instructions` only apply to the current request. When using `previous_response_id`, resend `instructions` each turn.

## Version prompts in code

OpenAI is deprecating reusable prompt objects. Prompt creation de-emphasized June 3, 2026; `v1/prompts` shuts down November 30, 2026.

For new work:
- Keep prompt builders in a small module near the feature they support.
- Use typed function arguments or schemas for dynamic values.
- Pass generated `instructions` and `input` directly to the Responses API.
- Add representative fixtures, tests, and evaluation checks before changing production prompts.
- Roll out prompt changes through your deployment system, using feature flags or configuration for staged releases.

## Message formatting with Markdown and XML

A `developer` message typically contains these sections in order:
- **Identity**: Purpose, communication style, and high-level goals.
- **Instructions**: Rules, what the model must/never do.
- **Examples**: Input/output pairs that teach behavior.
- **Context**: Additional relevant information (placed near the end).

## Prompt caching

Keep stable, repeated content near the beginning of the prompt and among the first API parameters to maximize prompt-cache savings.

## Few-shot learning

Include a diverse handful of input/output examples to steer the model toward a new task. Cover common and edge inputs. Keep examples consistent with stated rules.

## Include relevant context

Add context for proprietary data or specific resources. Be mindful of context window limits (varies by model, up to 1M tokens for newer GPT-4.1 models).

## Prompting reasoning models

Reasoning models work best with high-level goals and constraints; add detail only when evaluation results require it. GPT models need precise, explicit instructions.
