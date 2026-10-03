---
title: Realtime prompting guide
shortTitle: Realtime prompting
intro: Prompting techniques for gpt-realtime and the Realtime API, including voice agent structure, personality, tool use, and conversation flow.
---

## General tips

- Iterate relentlessly: small wording changes can make or break behavior.
- Prefer bullets over paragraphs.
- Guide with examples: the model closely follows sample phrases.
- Be precise: ambiguity or conflicting instructions degrade performance.
- Control language: pin output to a target language.
- Reduce repetition: add a Variety rule.
- Use capitalized text for emphasis on key rules.
- Convert non-text rules to text (e.g., "IF MORE THAN THREE FAILURES THEN ESCALATE").

## Prompt structure

```
# Role & Objective        — who you are and what "success" means
# Personality & Tone      — the voice and style to maintain
# Context                 — retrieved context, relevant info
# Reference Pronunciations — phonetic guides for tricky words
# Tools                   — names, usage rules, and preambles
# Instructions / Rules    — do's, don'ts, and approach
# Conversation Flow       — states, goals, and transitions
# Safety & Escalation     — fallback and handoff logic
```

## Role and objective

Pin identity explicitly. The model adheres tightly to role and objective when they are explicit.

## Personality and tone

- Set voice, brevity, and pacing so replies sound natural and consistent.
- For regulated domains, favor neutral precision.
- Use multi-emotion instructions sparingly and explicitly.

## Language constraint

Lock output to the target language. If the user speaks another language, politely explain support is limited.

## Reduce repetition

Add a Variety rule: do not repeat the same sentence twice; vary responses so it doesn't sound robotic. Whitelist must-keep phrases for legal/compliance/brand.

## Reference pronunciations

Keep a short list of phonetic hints for brand names, technical terms, and locations. Update as you hear errors.

## Alphanumeric pronunciations

Speak each character separately with separators (e.g., `4-1-5`). Confirm with the user and re-confirm after corrections. Use phonetic disambiguators for letters (e.g., "A as in Alpha").

## Instruction following

If instructions are conflicting, ambiguous, or unclear, performance degrades. Use an LLM to critique the prompt for ambiguity, lacking definitions, and unstated assumptions.

## No audio or unclear audio

Add custom instructions for how to behave when audio is unclear, background noise, or silence. Ask for clarification or repeat the last question.

## Background music or sounds

Add instructions to avoid unintended background music, humming, or sound-like artifacts.

## Tool selection

Ensure the prompt does not mention tools that are not available. Review available tools and system prompt for alignment.

## Tool call preambles

Add a short preamble before tool calls to mask latency (e.g., "I'm checking that now.").

## Tool calls without confirmation

For some use cases, remove confirmation loops before tool calls. Be careful not to make the model too eager.

## Tool call performance

As tools grow, explicitly guide when to use each tool and when not to. Add sequences of tool calls (after Tool A, call Tool B or C).

## Tool level behavior

Fine-tune behavior per tool: READ tools proactive, WRITE tools require confirmation.

## Tool output formatting

Wrap long or complex tool outputs in a small JSON envelope (e.g., `response_text` plus `require_repeat_verbatim`) so the output looks in-distribution and the realization constraint is machine-clear.

## Conversation flow

Break the interaction into phases with clear goals, instructions, and exit criteria. Prevents stalling, skipping steps, or jumping ahead.

## Sample phrases

Provide anchor examples for style, brevity, and tone. Warn: "DO NOT ALWAYS USE THESE EXAMPLES, VARY YOUR RESPONSES."

## Safety and escalation

Escalate on: safety risk, user requests human, severe dissatisfaction, 2 failed tool attempts, 3 consecutive no-match/no-input events, or out-of-scope requests.
