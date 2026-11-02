# Verification Checklist
- Confirm event triggers match the requested behavior.
- Confirm permission scope is least-privilege for `GITHUB_TOKEN`.
- Confirm secret handling and untrusted input boundaries are safe.
- Confirm runner selection is compatible with required tooling.
- Confirm caching keys and restore logic are deterministic.
- Confirm concurrency/cancellation behavior matches intent.
- Confirm environment and deployment protection rules are respected.
- Confirm all recommendations cite at least one relevant reference file.
