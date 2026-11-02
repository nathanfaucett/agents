# API Deployment Guidance

- For new OpenAI API work, manage prompts in application code, not reusable prompt objects.
- Keep a small prompt-builder module near the feature it supports.
- Use typed function inputs or schemas for dynamic values.
- Pass generated `instructions` and `input` directly to the Responses API.
- Pin a specific production model snapshot and evaluate behavior before changing the prompt or model.
- Roll out changes through normal deployment controls; use feature flags or configuration for staged releases.
- Reusable prompt objects are deprecated: creation was de-emphasized on June 3, 2026, and `v1/prompts` is scheduled to shut down on November 30, 2026. Use the OpenAI migration guide only for existing prompt-object integrations.
