# Review Workflow

Use this reference for the detailed parent-reviewer workflow.

## Output model

The parent reviewer may assemble a complete `json_review` object internally, then render markdown for the user.

### Stage 1: collect subagent partials

Each specialist subagent outputs a JSON partial following `templates/subagent-output.md`.

Key requirements:

- Score only the pillars owned by that specialist lens.
- Every finding must include `file` and `lines`.
- Use `suggested_fix` when the correct remediation is clear; otherwise say `No confident fix proposed`.

### Stage 2: assemble parent review data

Merge subagent partials into one internal `json_review` object following `templates/reviewer-output.md` and `references/json-review-schema.md`.

Assembly steps:

1. Collect non-null pillar scores from all partials and average duplicates.
2. Compute CQS and OIS from their non-null pillar scores.
3. Derive the grade from the score.
4. Map findings into blockers, action items, and recommendations while preserving `file` and `lines`.
5. Set status based on whether blockers or action items exist.
6. Count issues by severity.
7. Record reviewer notes covering lenses used, gaps, and key signal.

### Stage 3: draft the visible report

Use the internal review object to draft the visible markdown report and draft comments.

### Stage 4: render for the user

Return markdown only. Follow `templates/reviewer-output.md` for the final formatting.

## Required operating mode

- The parent reviewer owns final synthesis.
- Review the actual diff before launching subagents.
- Ensure one reviewer covers general code quality and correctness.
- Keep specialists independent and deduplicate overlapping findings in the parent synthesis.
- Preserve structured file and line metadata all the way through draft-comment generation.

## Large-project safety mode

Enable this mode when any of the following are true:

- More than 150 changed files.
- More than 10,000 changed lines.
- The diff is too large to review in one pass.

When safety mode is active:

- Use a two-pass review: triage first, then deep review of prioritized chunks.
- Keep chunks to about 25 files or 2,500 changed lines.
- Group chunks by concern such as auth, infra, UI, data model, or tests.
- State coverage clearly if the review is partial.

## Parallel review workflow

1. Establish the review target.
2. Discover available agents from `agents/*.agent.md`.
3. Gather baseline context from changed files and directly related code.
4. Choose specialist reviewers based on the diff.
5. Launch subagents in parallel with self-contained prompts.
6. Aggregate, deduplicate, and normalize findings in the parent reviewer.
7. Produce the final markdown review.

Prefer merge-base comparison against the remote tracking branch for branch reviews.

Illustrative command sequence:

```bash
default_branch="$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@')"
if [ -z "$default_branch" ]; then
  default_branch="$(git remote show origin | sed -n '/HEAD branch/s/.*: //p')"
fi

[ -n "$default_branch" ] || { echo "Could not determine default branch"; exit 1; }

git fetch origin "$default_branch" --quiet
git diff --stat "origin/$default_branch...HEAD"
git diff --name-only "origin/$default_branch...HEAD"
git diff --numstat "origin/$default_branch...HEAD"
```

If the default branch cannot be determined, ask instead of guessing.

## Branching logic

- Default reviewer set: code-and-QA, architecture, and security.
- Add principal or architecture depth for backend, infra, data, or architecture-heavy diffs.
- Add DevOps for CI, deployment, IaC, runtime config, or observability changes.
- Add UX for UI-heavy diffs.
- Add dedicated QA depth if critical behavior changed and test evidence is weak.
- Add performance review for hot paths, loops, throughput-sensitive code, or latency risk.

## Agent discovery

Build a roster from `agents/*.agent.md`:

1. List the files.
2. Parse each `name` and `description`.
3. Match lenses by keywords in the description.
4. Prefer the most specific match.
5. Use the inline prompt if no named agent matches.

## Parent reviewer checklist

- Diff and changed files inspected directly.
- Scale assessed and safety mode enabled when needed.
- Relevant surrounding code read.
- Agent roster built from the local `agents/` directory.
- Each subagent wave launched in a single batch.
- Findings categorized into Blockers, Bugs, Breaking Changes, Suggestions, Nitpicks.
- Every finding includes a project-relative file path and line information.
- Open questions used for anything that cannot be pinned to a line.
- Final review rendered from the internal review data and templates.