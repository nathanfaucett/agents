# Review Workflow

## Process

1. Identify the target and expected behavior.
2. Inspect the diff and relevant callers, tests, contracts, and configuration.
3. Select perspectives:
   - Always: code/QA.
   - Usually: architecture and security.
   - When relevant: UX, performance, data, or deployment.
4. Launch one subagent per perspective in parallel. Ask each to think about the code from that perspective; do not invoke named or repository-defined agents.
5. Check each reported issue against the code.
6. Merge duplicates and order by severity.

For branch reviews, prefer a merge-base diff against the remote default branch. Ask if the base cannot be determined safely.

## Finding standard

Report an issue only when it is actionable and supported by evidence. Include:

- Severity: Blocker, Bug, Breaking Change, or Suggestion.
- Project-relative file and line range.
- Impact.
- Fix, when clear.

Use Open Questions for risks that depend on missing context. Omit nitpicks unless requested.

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

Omit empty sections. If no findings remain after verification, say so and note any untested areas.