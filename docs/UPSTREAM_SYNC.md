# Upstream Sync

## Reference

Upstream: `miuuyy/codex-chatgpt-web`

The currently recorded reference revision is stored in `UPSTREAM_VERSION`.

## Policy

Keep browser/MCP/runtime deltas as small as possible so upstream fixes remain mergeable.

## Update procedure

1. Fetch upstream changes into a dedicated integration branch.
2. Read upstream browser, MCP, security and protocol changes before merging.
3. Update the source-level reuse map if boundaries changed.
4. Merge/rebase upstream with minimal conflict edits.
5. Run upstream regression tests first.
6. Run Hermes protocol/parser tests.
7. Run Hermes browser-only E2E.
8. Run Hermes tool-loop E2E.
9. Run cancellation/replay/concurrency regressions.
10. Update `UPSTREAM_VERSION` only after validation.
11. Record incompatible changes in a compatibility report.

## Divergence rule

Any intentional divergence in browser-worker, turn-broker, turn-execution, Responses parser, MCP transport or completion fencing must be documented with:
- upstream path;
- reason;
- local behavior;
- regression test;
- future merge risk.
