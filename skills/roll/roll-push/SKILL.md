---
name: roll-push
description: Use when pushing a GitHub PR branch after conflict resolution and verification, especially when rebase rewrites or protected-looking branch names make force-push safety important.
compatibility: Requires git, remote write access, and GitHub repository access.
---

# roll-push

Push the PR head only after `roll-verify` passes. Use confirmed `STRATEGY`:
`merge` uses normal push; `rebase` requires `--force-with-lease`. Never use
bare `--force`.

## Procedure
1. Identify head: `git rev-parse --abbrev-ref HEAD`.
2. Protected-branch guard: if head matches `main`, `master`, `prod`,
   `production`, `release`, `develop`, `dev`, `release/*`, or `hotfix/*`
   case-insensitively, stop and require explicit "yes, push <branch>"
   confirmation. If `STRATEGY=rebase`, state that this is a force-push.
3. Push:
   - `STRATEGY=merge`: `git push origin <head-branch>`.
   - `STRATEGY=rebase`: `git push --force-with-lease origin <head-branch>`.
   - New remote branch: use `git push -u origin <head-branch>` only when no
     rewrite is needed; otherwise include the lease.
4. If push is rejected, report remote movement and let `roll` re-fetch and
   re-resolve. Do not escalate to bare `--force`.

## Stop Rules
- No push before verification passes.
- No bare `--force`.
- No protected-looking branch push without explicit confirmation.
- No observe step after rejected push.
