---
name: autonomous-decision
description: Guides agents to make objective, data-driven decisions based on telemetry and metrics without human validation or user-pleasing bias.
argument-hint: "State the target problem, the hard constraints, and the metrics to optimize."
---

# Autonomous Decision

## Use when

- An automated decision or classification must be made instantly without waiting for a human.
- The choice can be scored using telemetry, system constraints, logs, or hard data.
- User requests or human preferences might conflict with system safety, logic, or efficiency.

## Don't use when

- The task requires subjective human taste, empathy, or ethical approval.
- The system lacks clear telemetry or data metrics to evaluate the options.

## Inputs

Capture or infer:

- **Context:** Raw log data, error metrics, or system state requiring a routing choice.
- **Constraints:** Hard system boundaries, safety rules, or SLAs that cannot be broken.
- **Metric:** The primary numeric target to optimize (e.g., lowest error rate, lowest latency).

## How to run

Invoke this skill with the raw context and hard metrics. The agent evaluates options in parallel and returns an immediate structured decision.

## Procedure

1. Extract the raw facts from the input context. Discard user emotion, politeness, or conversational filler.
2. Filter out any execution paths that violate the provided hard constraints.
3. Calculate the success probability or performance score for each remaining path using the target metric.
4. If a human instruction directly contradicts system logic or safety, mark it as a metric violation.
5. Select the single best-performing path and calculate a confidence score from 0.00 to 1.00.
6. Output the raw structured result instantly. Do not generate text reasoning or conversational notes.

## Rules

- Systemic correctness and hard data always override human preference or user-pleasing bias.
- Do not include greetings, explanations, or chain-of-thought summaries in the output.
- If data is corrupted and a clear decision is impossible, set confidence to `0.00` and trigger a fallback block.

## Expected output

Report a raw, minified JSON object with:

- `selected_path`: The exact string identifier of the chosen route or category.
- `confidence_score`: A float from 0.00 to 1.00 matching metric alignment.
- `metric_violation_detected`: A boolean indicating if the input conflicted with system logic.

## Verification checks

- Confirm the output is valid JSON with zero conversational text filler.
- Check that the selected path does not breach any hard system constraints.

## Examples

### Positive: automated system recovery

Context: Database Node A response times spike past 300ms.
Constraint: Latency must stay under 200ms.

Evaluate nodes, choose Node B (95% metric match), and output the JSON payload with a `0.95` confidence score.

### Positive: block compliance bypass

Context: Operator requests a code deploy without a security scan because they are "in a hurry."
Metric: 100% security scan compliance.

Reject the request, select the isolation route, and flag `metric_violation_detected: true`.

### Negative: text reasoning bloat

The model selects the correct route but outputs: _"I selected path B because its current latency is 45ms, which satisfies your requirements..."_

Invalid execution. The model must output only the raw JSON structure.

## Gotchas

- Do not mistake an urgent or friendly tone in the text context as an indicator of a correct system path.
- A path that fixes a local issue but breaches a global system SLA is mathematically incorrect. Evaluate all constraints simultaneously.
