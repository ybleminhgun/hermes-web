# Release Validation

## CI-verifiable

- build
- lint
- typecheck
- unit tests
- schema validation
- parser fixtures
- protocol fixtures
- replay/idempotency tests
- capability/identity tests
- deterministic concurrency tests where possible

## Account-bound/manual integration

These cannot be inferred solely from CI and must be explicitly recorded:

- authenticated ChatGPT Web session;
- model/effort selector compatibility;
- live browser submission/extraction;
- configured ChatGPT MCP connector;
- end-to-end Hermes tool round;
- cancellation with a live browser turn;
- Windows packaged launcher smoke.

## Release result

A release report must list every required gate as PASS, FAIL, BLOCKED or NOT_TESTED. A release cannot be called production-ready while mandatory account-bound E2E checks are merely NOT_TESTED.

## Required matrix

BUILD, LINT, TYPECHECK, UNIT, PARSER, PROTOCOL, BROWSER_SMOKE, HERMES_BROWSER_ONLY_E2E, HERMES_TOOL_E2E, MULTI_ROUND_E2E, CANCELLATION, STALE_CAPABILITY, REPLAY, PARALLEL_TOOL, APPROVAL, CONTEXT_EPOCH, DELEGATION, PACKAGE_SMOKE, DEPENDENCY_AUDIT.
