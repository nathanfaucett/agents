# Procedure

1. **Define the target behavior**
   - Write a one-sentence objective for the prompt.
   - Define success as observable output criteria.

2. **Set instruction hierarchy**
   - Put high-priority behavior first.
   - Separate required rules from preferences.
   - State what the model must do and must not do.
   - Put reusable application rules in the Responses API `instructions` parameter or a `developer` message; they take priority over `user` input.
   - Do not rely on `instructions` from a prior turn when using `previous_response_id`; send them again.

3. **Draft the prompt with consistent formatting**
   - Use clear section headers in this order when useful: `Identity`, `Instructions`, `Examples`, `Context`.
   - Keep instructions explicit and testable.
   - Use placeholders for dynamic values, for example `{{customer_name}}`.
   - Keep stable, repeated content near the beginning to improve prompt-cache savings.

4. **Strengthen formatting discipline**
   - Use bullet lists for constraints and output rules.
   - Use XML-style delimiters for example I/O blocks when helpful.
   - Avoid long paragraphs that hide requirements.

5. **Add examples that teach behavior**
   - Use a diverse handful of examples when they materially improve reliability, especially for strict formats.
   - Cover common and edge inputs.
   - Keep examples consistent with stated rules.

6. **Add output contract checks**
   - Specify exact format expectations (fields, ordering, allowed labels).
   - Add explicit negative constraints to reduce drift.

7. **Revise for clarity and brevity**
   - Remove redundant instructions.
   - Replace vague wording with measurable requirements.
   - Keep only context that changes model decisions.
   - For GPT models, use precise instructions. For reasoning models, begin with higher-level goals and add detail only when evaluation results require it.

8. **Add operational readiness notes when the prompt will be used as a skill artifact**
   - Include when to use, how to run, expected outputs, and gotchas.
   - Add explicit verification steps and output checks for brittle multi-step workflows.
