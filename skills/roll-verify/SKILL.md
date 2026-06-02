---
name: roll-verify
description: Use after resolving conflicts and before pushing a PR branch to verify the resolution with project tests or user-supplied checks.
compatibility: Requires shell access and the project's verification tooling when available.
---

# roll-verify

Verify before `roll-push`; a clean merge is not sufficient.

## Procedure
1. Discover verification from project evidence: `Makefile`, `package.json`,
   `pyproject.toml`, `tox.ini`, `Cargo.toml`, `go.mod`, or
   `.github/workflows/*.yml`.
2. Prefer the native command found there: `make test`, package-manager test
   script, `pytest`/`tox`, `cargo test`, or `go test ./...`.
3. If no command is evidenced, ask for verification instructions or a goal. Do
   not invent a command or skip silently.
4. Run verification and report pass/fail with relevant output.
5. On failure, return fail and block `roll-push`; `roll` decides the next step.

## Stop Rules
- No push after failed verification.
- No invented test command.
- No silent verification skip.
