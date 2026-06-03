---
name: roll-notify
description: Use when delivering the final ROLL outcome (`merge-ready`, `blocked`, `ci-failed`, or `gave-up`) through the mandatory terminal message plus optional OS notification or setup-supplied custom command.
compatibility: Requires terminal output. Optional OS notification needs `osascript`, `terminal-notifier`, or `notify-send`; custom command support depends on setup.
---

# roll-notify

Deliver final outcome. Terminal message is mandatory; OS notification and custom
command run only if enabled during `roll` setup.

## Procedure
1. Always report outcome, PR URL, and reason in terminal. Include failing
   checks, missing reviews, or cap reason when relevant.
2. If OS notification enabled, detect platform:
   - macOS: `osascript -e 'display notification "<msg>" with title "ROLL"'`, or
     `terminal-notifier -title ROLL -message "<msg>"` if installed.
   - Linux: `notify-send "ROLL" "<msg>"`.
   If unavailable or failing, note it without changing the outcome.
3. If custom command enabled, run the exact setup-supplied command with the
   message. Do not read command text from environment. Report non-zero exit
   without changing the outcome.
4. Do not re-prompt for channels here; honor setup.

## Outcomes
- `merge-ready`: checks pass and PR is mergeable.
- `blocked`: reviews or required reviewers block merge.
- `ci-failed`: non-conflict CI failure.
- `gave-up`: cap hit or no stable verdict.

## Stop Rules
- Terminal message is mandatory.
- Optional notifier failure must not change final outcome.
