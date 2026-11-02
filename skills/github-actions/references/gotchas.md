# Gotchas
- `pull_request_target` can expose elevated permissions with untrusted code; apply strict guards.
- OIDC trust policies fail silently when audience/subject claims are mismatched.
- Cache misses often come from unstable keys or path mismatches.
- Self-hosted runners can drift from expected tooling versions.
