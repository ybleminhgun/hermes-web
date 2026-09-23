# Troubleshooting

This file starts as a framework and must be expanded from real failures.

## Diagnose by layer

### Hermes/provider
Check provider discovery, `codex_responses` request shape, runtime bearer token, session/turn metadata and cancellation.

### Responses/parser
Use captured fixtures. Do not debug browser behavior until parser normalization is proven correct.

### Browser
Check authenticated profile, ChatGPT UI compatibility, prompt acceptance, model/effort selection, logical response-turn detection and completion state.

### MCP/broker
Check connector availability, capability token, active turn, advertised tool allowlist, call ID and outstanding invocation state.

### Hermes tool runtime
Check that the call reaches normal Hermes tool dispatch rather than direct bridge execution.

## Common safety rule

Never fix a failure by disabling stale-turn checks, capability validation, runtime authentication or completion fencing. First isolate the mismatched state and add a regression test.
