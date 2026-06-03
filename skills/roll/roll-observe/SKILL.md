---
name: roll-observe
description: Use after pushing a GitHub PR branch to watch Actions while polling mergeability. Returns capped verdicts and aborts early when base movement creates conflicts.
compatibility: Requires GitHub CLI (`gh`), GitHub repository access, and a coding-agent host that can run shell commands.
---

# roll-observe

Return one verdict: `green`, `failed`, `conflicted`, `review-blocked`, or
`gave-up`. Merge-ready requires passing checks, clean mergeability, and required
reviews.

## Default Shape
Run two concurrent processes:
1. Checks watcher: `gh pr checks <pr> --watch`.
2. Conflict poller every `CONFLICT_POLL_INTERVAL` default 10 min:
   `gh pr view <pr> --json mergeable,mergeStateStatus`.

If any poll returns `mergeable=CONFLICTING` or `mergeStateStatus=DIRTY`, kill
the watcher and return `conflicted`. If the watcher exits first, reconcile.

## Fallbacks
Use the best supported shape:
1. Host scheduler tick: each tick reads checks plus mergeability JSON.
2. Two-process watch plus poller: default.
3. Foreground poll loop: read checks and mergeability each interval. Do not use
   a bare blocking `--watch`; it misses early conflicts.

## Reconcile
After any watch exit, read current truth:
```
gh pr checks <pr>
gh pr view <pr> --json mergeable,mergeStateStatus,reviewDecision,statusCheckRollup
```

- Terminal checks: decide verdict from current truth.
- Pending/in_progress checks: watch likely died early; re-watch with backoff,
  capped by `MAX_REWATCH` default 5.
- `gh pr checks` exits 8: treat as no checks to wait for.

## Caps
- Honor remaining shared `MAX_WALL` from `roll`; do not start a new timer.
- On `MAX_REWATCH` or wall-clock cap hit, return `gave-up`.
- Avoid polling so fast that `gh`/API rate limits become likely.

## Verdict Rules
- `green`: checks pass, mergeability clean, required reviews met.
- `failed`: required check failed.
- `conflicted`: `mergeable=CONFLICTING` or `mergeStateStatus=DIRTY`.
- `review-blocked`: reviews or required reviewers block merge.
- `gave-up`: cap hit or no stable verdict.
