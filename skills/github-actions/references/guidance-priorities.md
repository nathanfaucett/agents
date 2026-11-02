# Guidance Priorities

When producing output, prioritize this order:
1. Correctness against workflow syntax and event behavior.
2. Security posture (secrets handling, token permissions, OIDC, untrusted inputs).
3. Runner compatibility and cost/performance tradeoffs.
4. Reliability (caching strategy, cancellation controls, retries, concurrency).
5. Maintainability (reusable workflows, clear naming, minimal duplication).
