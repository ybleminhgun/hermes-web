# Protocol

## V1 protocol choice

Use OpenAI Responses-compatible HTTP between Hermes and the local bridge. Prefer Hermes `codex_responses` mode.

## Endpoints

```text
POST /v1/responses
GET  /v1/models
GET  /health
GET  /diagnostics
```

## Authentication

Trusted Hermes calls require a runtime bearer token. The daemon binds to loopback by default.

## Trusted metadata envelope

The exact serialization is to be finalized during G1/G2 after capturing a real Hermes request, but the semantic contract is fixed by `contracts/identity.schema.json` and `contracts/environment.schema.json`.

Prompt/user text is never an authority source.

## Main turn

`purpose=agent_turn`.

A logical turn may span multiple HTTP/model rounds. `turn_id` remains stable; `round_id` changes.

## Auxiliary request

`purpose=auxiliary`.

Auxiliary work must use isolated execution state and no main-turn capability token. MCP is disabled by default.

## Tool conversion

Accepted ChatGPT MCP requests are translated to Responses `function_call` output for Hermes. Hermes native execution returns `function_call_output` on a continuation round.

## Replay

- identical replay: idempotent where safe;
- same call ID with different content: conflict;
- late/stale call: reject;
- retired capability: reject.

## Completion

Final browser output is committed only after browser completion and tool activity are settled under the current activity generation/revision.
