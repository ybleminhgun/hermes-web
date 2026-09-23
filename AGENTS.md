# AGENTS.md — Mandatory Implementation Rules

This file is authoritative for coding agents working in this repository.

## Mission

Replace the outer Codex agent/runtime role used by `codex-chatgpt-web` with Hermes Agent while preserving the upstream browser/MCP runtime as much as possible.

## Non-negotiable invariants

1. **MAIN_AGENT_LLM_COUNT = 1.** ChatGPT Web is the main reasoning model.
2. Do not insert a second reasoning/planning LLM between Hermes and ChatGPT Web.
3. Hermes owns durable session/context, memory, approvals, delegation and actual tool execution.
4. The bridge is deterministic transport/runtime glue, not an autonomous agent.
5. ChatGPT may request tools but must never directly execute Hermes local capabilities.
6. Only tools advertised by Hermes for the active model request may be invoked.
7. Each Full Mode turn uses an ephemeral capability token. Retired tokens never regain authority.
8. Trusted execution metadata must not be reconstructed from user-authored prompt text.
9. Provider identity is sticky for all rounds of one active logical turn.
10. Any uncertainty in authority, turn identity, completion, replay or browser state must fail closed.

## Upstream-first rule

Treat `miuuyy/codex-chatgpt-web` as the reference upstream.

Prefer KEEP, PATCH_MINIMAL and GENERICIZE. Avoid wholesale rewrites, mechanical Codex→Hermes renames, or replacing browser/MCP logic without evidence.

Especially do **not** rewrite the browser worker, turn broker, turn execution, Responses parser or completion fencing unless a failing compatibility test proves it is necessary.

## Tool execution path

Required:

```text
ChatGPT MCP request
  -> bridge broker
  -> Responses function_call
  -> Hermes AIAgent
  -> Hermes native tool dispatcher
  -> function_call_output
  -> bridge
  -> same ChatGPT browser turn
```

Forbidden:

```text
ChatGPT MCP -> bridge -> direct fs/shell/browser execution
```

## Context ownership

Hermes owns durable truth. The bridge owns browser representation only. Do not treat ChatGPT browser conversation as persistent Hermes history.

## Implementation workflow

- Work one gate at a time.
- Before editing, inspect current behavior and tests.
- Keep commits small and phase-scoped.
- Preserve upstream tests where applicable.
- Add Hermes-specific fixtures/regressions before broad refactors.
- Never delete validation or weaken authority checks merely to make tests pass.
- Report gate status as one of: PASS / FAIL / BLOCKED / NOT_TESTED.

## Mandatory reading order

1. `docs/ANTIGRAVITY_EXECUTION_PLAN.md`
2. `docs/ARCHITECTURE.md`
3. `docs/HERMES_CONTRACT.md`
4. `docs/SECURITY_MODEL.md`
5. `docs/TEST_PLAN.md`
6. `docs/UPSTREAM_REUSE_MAP.md`
