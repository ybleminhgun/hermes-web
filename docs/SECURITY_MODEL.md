# Security Model

## Trust boundaries

Separate these boundaries:
1. Hermes runtime
2. local bridge daemon
3. Electron/browser profile
4. authenticated ChatGPT Web session
5. MCP connector/tunnel
6. local filesystem/tool runtime
7. repository and web content
8. user-authored prompt content

Repository files, web pages, tool outputs and prompt text are untrusted input.

## Authority chain

```text
User
 -> Hermes decides active tools/permissions
 -> Bridge binds them to an ephemeral turn capability
 -> ChatGPT may request only those capabilities
 -> Hermes performs actual execution
```

ChatGPT never self-authorizes a local capability.

## Local daemon

- bind to loopback only by default;
- require a random runtime bearer token for trusted Hermes calls;
- never expose browser cookies/storage/session secrets through diagnostics;
- reject conflicting trusted metadata.

## Turn capability

Capability tokens must be unpredictable, scoped to one active logical turn, revocable, and retired on completion/cancellation/fatal invalidation. A retired token can never be reactivated.

## Prompt boundary

Never derive cwd, workspace roots, permission profile, tool allowlist, or session/turn identity from user-authored prompt text.

## Tool execution

Bridge must never bypass Hermes native tool policy by directly performing filesystem/shell/browser actions on behalf of MCP.

## Completion safety

Browser visual completion is not sufficient. Finalization requires settled tool activity and an unchanged activity revision/generation.

## Replay

Identical replay may be treated idempotently where safe. Conflicting replay is a hard error.

## Diagnostics redaction

Redact Authorization headers, browser cookies/storage, tunnel credentials, capability tokens, secret environment values and raw authentication artifacts.

## Fail-closed rule

Unknown authority, identity or completion state results in explicit failure, never guessed success.
