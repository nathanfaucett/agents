# Examples & Appendix

This file holds long examples and helpful scripts moved from `SKILL.md` to keep the main skill concise.

## Git diff helper (example)
```bash
default_branch="$(git remote show origin | sed -n '/HEAD branch/s/.*: //p')"
if [ -z "$default_branch" ]; then
   default_branch="$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@')"
fi
[ -n "$default_branch" ] || { echo "Could not determine default branch"; exit 1; }

git fetch origin "$default_branch" --quiet
git diff --stat "origin/$default_branch...HEAD"
git diff --name-only "origin/$default_branch...HEAD"
git diff --numstat "origin/$default_branch...HEAD"
```

## Extended checklist and prompts
Long-form checklist items, per-lens prompt templates, and other extended guidance were moved here. For the original, full-length content see `SKILL.md.bak`.
