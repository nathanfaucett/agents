# Review Workflow

## Process

1. Identify the target and expected behavior.
2. Inspect the diff, callers, tests, contracts, and configuration.
3. Select perspectives:
   - Always: code/QA and design.
   - Usually: architecture and security.
   - When relevant: UX, performance, data, or deployment.
4. Run one parallel subagent per perspective; do not invoke repository-defined agents.
5. Verify, deduplicate, and prioritize findings.

**Design** asks whether the change is the smallest suitable solution to a demonstrated problem. Flag only a concrete simpler existing path, unneeded abstraction/configuration/dependency, or a wrong seam; otherwise raise an Open Question. Keep it separate from scope creep: scope creep exceeds the spec; design challenges the specified approach.

For branch reviews, prefer a merge-base diff against the remote default branch. Ask if the base is unsafe to determine.

## Findings

Report only actionable, evidence-backed issues with severity (Blocker, Bug, Breaking Change, or Suggestion), project-relative location, impact, and a fix when clear. Put missing-context risks in Open Questions; omit nitpicks.

## Output

```markdown
## Review

### Blockers

- **Title** — `path/to/file.ts:L10-L15`
  Impact and fix.

### Bugs

### Breaking Changes

### Suggestions

### Open Questions
```

Omit empty sections. If none remain, say so and note untested areas.
