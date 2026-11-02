# Reference-Driven Workflow

Use this sequence on every run:
1. Identify the task type: authoring, review, debugging, migration, or hardening.
2. Start from the category index files to find the canonical reference page.
3. Read the focused page for the exact topic.
4. Produce output using the output contract template in this skill.
5. Run verification checks before finalizing.

## Reference Indexes
Start from these files:
- `references/index.md`
- `references/workflows-and-actions/index.md`
- `references/runners/index.md`
- `references/security/index.md`
- `references/github-actions-importer/index.md`

## Reference Outline

### Workflows And Actions
- Author or review workflow YAML.
- Choose correct events, expressions, and contexts.
- Configure caching, environments, runners, and secrets.
- Apply OIDC and secure-use best practices.
- Understand platform limits and Actions Importer behavior.
- Workflow syntax: `references/workflows-and-actions/workflow-syntax.md`
- Trigger events: `references/workflows-and-actions/events-that-trigger-workflows.md`
- Workflow commands: `references/workflows-and-actions/workflow-commands.md`
- Variables: `references/workflows-and-actions/variables.md`
- Expressions: `references/workflows-and-actions/expressions.md`
- Contexts: `references/workflows-and-actions/contexts.md`
- Deployments and environments: `references/workflows-and-actions/deployments-and-environments.md`
- Dependency caching: `references/workflows-and-actions/dependency-caching.md`
- Reusable workflow configs: `references/workflows-and-actions/reusing-workflow-configurations.md`
- Action metadata syntax: `references/workflows-and-actions/metadata-syntax.md`
- Workflow cancellation behavior: `references/workflows-and-actions/workflow-cancellation.md`
- Dockerfile support: `references/workflows-and-actions/dockerfile-support.md`

### Runners
- GitHub-hosted runners: `references/runners/github-hosted-runners.md`
- Larger runners: `references/runners/larger-runners.md`
- Self-hosted runners: `references/runners/self-hosted-runners.md`

### Security
- Secure use reference: `references/security/secure-use.md`
- Secrets reference: `references/security/secrets.md`
- OIDC reference: `references/security/oidc.md`

### Limits
- Actions limits: `references/limits.md`

### GitHub Actions Importer
- Importer reference: `references/github-actions-importer/index.md`
- Supplemental arguments and settings: `references/github-actions-importer/supplemental-arguments-and-settings.md`
- Custom transformers: `references/github-actions-importer/custom-transformers.md`
