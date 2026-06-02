---
name: roll-resolve
description: Use when a GitHub PR branch conflicts with its base and the agent must integrate the base without silently mis-merging behavior.
compatibility: Requires git, repository write access, and project tooling needed to regenerate generated conflict artifacts such as lockfiles.
---

# roll-resolve

Integrate base using the confirmed `STRATEGY`. Resolve only conflicts that can
preserve both sides' intent. For semantic conflicts, stop and hand off.

## Procedure
1. Fetch and integrate base:
   - `rebase`: `git fetch origin <base> && git rebase origin/<base>`
   - `merge`: `git fetch origin <base> && git merge origin/<base>`
   Do not choose strategy here; `roll` already confirmed it.
2. Classify conflicts:
   - Auto-resolve: lockfiles, generated files, import ordering, and strictly
     non-overlapping hunks where keeping both preserves intent.
   - Take-over: overlapping logic edits, semantic conflicts, or any resolution
     requiring behavior choice or intent guessing.
3. Auto-resolve only safe conflicts. Regenerate lockfiles when practical; state
   what changed and why intent is preserved.
4. For take-over conflicts, stop. Show file/hunk and why it is unsafe; let the
   user resolve it or give explicit per-file instructions. Wait for completion.
5. When conflict-free, continue: `git rebase --continue` or commit the merge.
6. Output a clean tree ready for `roll-verify`, or a handoff list of unresolved
   take-over conflicts.

## Stop Rules
- Do not edit semantic conflicts without explicit per-file instruction.
- Do not treat handoff as failure.
- Do not hand-edit lockfiles when regeneration is available.
