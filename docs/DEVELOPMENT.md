# Development

## Required workflow

1. Read `AGENTS.md`.
2. Identify the active gate in `docs/ANTIGRAVITY_EXECUTION_PLAN.md`.
3. Inspect current tests and behavior before editing.
4. Add/adjust a deterministic fixture or regression test where possible.
5. Make the smallest implementation change.
6. Run the gate-specific checks.
7. Record PASS/FAIL/BLOCKED/NOT_TESTED.
8. Commit only one coherent gate-sized change set.

## Branches

Recommended:
- `main`: validated project state;
- `upstream-sync/*`: temporary upstream integration;
- `phase/gX-*`: implementation work for a gate;
- `fix/*`: isolated regression fixes.

## Commit expectations

Commit messages should describe behavior, not just files. Keep protocol changes, browser changes and security changes easy to audit.

## No silent architecture changes

If implementation reality contradicts the plan:
1. preserve working code;
2. reproduce the mismatch;
3. document it;
4. propose the smallest change;
5. add a test;
6. only then alter architecture.
