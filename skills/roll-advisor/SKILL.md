---
name: roll-advisor
description: Use after resolving a PR conflict when optional advice is wanted to reduce future conflict risk. Every suggestion must preserve exactly what the PR accomplishes.
compatibility: Requires git history access. Interactive selection UI is optional.
---

# roll-advisor

Optional, gated advice after `roll-resolve`. Skip when advisor mode is `off`.
Use interactive UI for skip/apply choices when available; otherwise ask concise
text questions. Never apply advice without explicit approval.

Invariant: each suggestion must preserve the PR's original behavior and scope.
If a suggestion changes what the PR accomplishes, label it goal-altering and
require explicit approval before treating it as an option.

## Procedure
1. Inspect conflicted files, resolved hunks, and history:
   `git log --oneline -20 -- <file>` and `git shortlog -sn -- <file>`.
2. Identify conflict cause: hot file, weak boundary, broad PR, base churn,
   commit order, strategy mismatch, etc.
3. Offer one up-front "ignore / skip all" choice. If chosen, drop all advice for
   this run.
4. Suggest valid conflict-reducing actions: split PR, reorder/squash commits,
   narrow scope, extract contended code, switch merge/rebase strategy next time,
   or coordinate around base churn.
5. For each suggestion, state how it preserves the original goal.
6. Apply only approved suggestions. Otherwise continue with the existing
   resolution; advice is non-blocking.

## Stop Rules
- Do not edit before approval.
- Do not present behavior-changing work as normal advice.
- Do not re-ask after the user skips all advice.
