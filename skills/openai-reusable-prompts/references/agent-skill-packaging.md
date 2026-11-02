# Agent Skill Packaging

Apply these when the prompt template is packaged as an agent skill, not as an OpenAI API prompt object.

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
