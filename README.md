# Hermes Web

Hermes Web is a greenfield integration project that reuses the proven architecture of `miuuyy/codex-chatgpt-web` while replacing the outer Codex agent/runtime role with Hermes Agent.

## Mission

The target runtime relationship is:

```text
User
  ↓
Hermes AIAgent
  ↓  codex_responses
Hermes Web local bridge
  ↓
ChatGPT Web
  ↕  MCP tool requests
Hermes Web capability broker
  ↓  Responses function_call
Hermes AIAgent
  ↓
Hermes native tool runtime
```

### Core ownership

- **Hermes** owns agent orchestration, durable session/context, memory, approvals, delegation and actual tool execution.
- **ChatGPT Web** is the primary reasoning model.
- **Hermes Web bridge** is deterministic transport/runtime glue: Responses API, browser automation, MCP reverse path, turn capability validation, streaming and lifecycle.
- **No intermediary reasoning LLM** belongs between Hermes and ChatGPT Web.

## Implementation strategy

This repository follows an **upstream-first, minimal-delta, test-gated** strategy.

The implementation MUST preserve as much of `codex-chatgpt-web` as practical, especially:

- browser worker and ChatGPT session automation;
- Electron persistent profile/session ownership;
- Responses parser and SSE translation;
- turn execution and capability broker;
- MCP transport and tool-round continuation;
- completion fencing and cancellation behavior.

Codex-specific identity, environment, installer/configuration and outer-runtime assumptions are replaced with Hermes equivalents behind a dedicated harness boundary.

## Start here

Antigravity or any implementation agent MUST read these files before editing source:

1. `AGENTS.md`
2. `docs/ANTIGRAVITY_EXECUTION_PLAN.md`
3. `docs/ARCHITECTURE.md`
4. `docs/HERMES_CONTRACT.md`
5. `docs/SECURITY_MODEL.md`
6. `docs/TEST_PLAN.md`

Do not skip phase gates.

## Initial milestones

- G0 — upstream baseline and reuse audit
- G1 — capture real Hermes `codex_responses` fixture
- G2 — parser compatibility
- G3 — harness abstraction
- G4 — Hermes provider plugin
- G5 — browser-only Hermes → ChatGPT Web → Hermes
- G6 — single tool round through MCP
- G7 — multi-round tool loop
- G8 — error/retry/idempotency
- G9 — parallel tools and approvals
- G10 — context/session ownership
- G11 — Hermes memory/delegation
- G12 — auxiliary isolation
- G13 — launcher
- G14 — Windows application
- G15 — security hardening
- G16 — release validation

## Status

**Specification/bootstrap stage. No production implementation yet.**

## Upstream

Reference implementation: `https://github.com/miuuyy/codex-chatgpt-web`

The project must preserve upstream license/attribution requirements for reused code.
