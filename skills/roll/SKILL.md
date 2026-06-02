---
name: roll
description: "Drive a GitHub pull request from conflicts or failing checks to merge-ready status. Use when the agent should confirm the PR target, resolve against base, verify, push, observe Actions and mergeability, loop safely, and notify the result."
compatibility: Requires git, GitHub CLI (`gh`), GitHub repository access, and a coding-agent host that can run shell commands. Interactive selection UI is optional.
---

# roll

Capped PR loop: resolve conflicts, optionally advise, verify, push,
observe CI plus mergeability, repeat until merge-ready or stopped.

## Interaction
For fixed-choice questions, prefer available interactive UI (`request_user_input`,
host selection prompts, etc.). Use it for setup groups, observe confirmations,
retry/stop decisions, and advisor gates.

If interactive UI is unavailable, ask concise text questions and include this
hint: "Interactive selection UI is unavailable in this agent/session. To enable
it, use a coding-agent host or mode that exposes `request_user_input` or
equivalent selection prompts; otherwise answer these choices in text."

Setup UI must include every group below: target/integration, advisory,
notification, and observe settings. Do not omit observe settings in previews or
real runs. Do not re-prompt for settings already confirmed.

## Setup
Before mutating a branch:

**Target + integration**
1. Detect target: `gh pr view --json number,headRefName,baseRefName,url`. Show
   PR, head, base, and URL; require confirmation or correction. If no PR exists,
   stop and ask whether to create one or use another target.
2. If head looks protected (`main`, `master`, `prod`, `production`, `release`,
   `develop`, `dev`, `release/*`, `hotfix/*`), require explicit extra
   confirmation. `roll-push` must re-check.
3. Ask `STRATEGY`: `merge` or `rebase`. Detect repo convention and propose a
   default, but never choose silently. `merge` uses normal push; `rebase`
   rewrites history and later requires `--force-with-lease`.

**Advisory settings**
4. Ask advisor mode: `off` default, `on`, or `ask-each-time`.

**Notification settings**
5. Ask notify channels: terminal always on; optional OS notification; optional
   custom command supplied now, not read from environment.

**Observe settings**
6. Set `CONFLICT_POLL_INTERVAL`, default 10 min.
7. Set `CONFIRM_EACH_OBSERVE`, default off. If on, pause after every observe
   verdict before acting.

## Loop
Caps: `MAX_ITERS` default 10; `MAX_WALL` default 60 min shared across the whole
run, including `roll-observe`. Every re-loop consumes an iteration.

1. If conflicted, run `roll-resolve`. If advisor enabled, run `roll-advisor`.
   Run `roll-verify`; on fail, do not push. Ask whether to retry via a new
   iteration.
2. Run `roll-push`. If rejected or failed, do not observe and never escalate to
   bare `--force`; loop back to fetch/resolve.
3. Run `roll-observe`. If `CONFIRM_EACH_OBSERVE` is on, show the verdict and
   wait for approval before acting.
4. Verdict handling:
   - `green`: success.
   - `conflicted`: loop.
   - `failed`: stop, report failing checks/logs, ask; do not auto-fix CI.
   - `review-blocked`: stop, report missing reviews/requirements.
   - `gave-up`: stop.
5. If any cap is hit, stop as `gave-up`.

## Finish
Run `roll-notify`:

| Loop result | Notify outcome |
|---|---|
| `green` | `merge-ready` |
| `failed` | `ci-failed` |
| `review-blocked` | `blocked` |
| `gave-up` or cap hit | `gave-up` |

## Dispatch
`roll-resolve`: conflicts. `roll-advisor`: optional future-conflict advice.
`roll-verify`: validation. `roll-push`: remote update. `roll-observe`: checks
and mergeability. `roll-notify`: final report.

## Stop Rules
- No branch mutation before target confirmation.
- No protected-looking force-push without explicit confirmation.
- No advisor when mode is `off`.
- No auto-fix for non-conflict CI failures.
- No uncapped loop.
