# Antigravity Execution Plan

## Objective

Build a new working repository named `hermes-web` using `miuuyy/codex-chatgpt-web` as the upstream/reference implementation, but replace the outer Codex agent/runtime role with Hermes Agent.

Do not redesign the browser or MCP runtime unless compatibility evidence requires it.

## Mandatory architecture

```text
User
 -> Hermes AIAgent
 -> codex_responses
 -> local Hermes Web bridge
 -> ChatGPT Web
 -> optional MCP tool request
 -> TurnBroker/capability validation
 -> Responses function_call
 -> Hermes AIAgent
 -> Hermes native tool runtime
 -> function_call_output
 -> same ChatGPT browser turn
 -> final response
 -> Hermes
```

## Core rules

- ChatGPT Web is the single main reasoning model.
- No intermediary reasoning LLM.
- Hermes owns durable state and actual tool execution.
- Bridge owns transport/browser/MCP/capability lifecycle only.
- Preserve upstream browser worker, MCP path, Responses parser and completion fencing wherever possible.
- Do not mass-rename Codex symbols before end-to-end compatibility is proven.
- Every mandatory phase is test-gated.

---

# Gate G0 — Upstream baseline

## Tasks

1. Record exact upstream commit/tag.
2. Import or vendor upstream code in a way that preserves history/attribution where practical.
3. Build upstream unchanged.
4. Run lint/typecheck/unit tests.
5. Run browser-only smoke if environment permits.
6. Run Full Mode/MCP smoke if environment permits.
7. Produce a source-level reuse map.

## Deliverables

- `UPSTREAM_VERSION`
- `docs/UPSTREAM_REUSE_MAP.md` updated with exact paths
- `reports/G0_BASELINE.md`

## PASS

- baseline builds;
- mandatory upstream tests pass;
- every major upstream module classified KEEP/PATCH_MINIMAL/GENERICIZE/REWRITE/REMOVE/DEFER.

No Hermes functional changes before G0 passes.

---

# Gate G1 — Hermes Responses discovery

## Goal

Determine the real wire compatibility between Hermes `codex_responses` and upstream Codex requests.

## Tasks

Capture real fixtures:

- `tests/fixtures/hermes-responses-request.json`
- `tests/fixtures/codex-responses-request.json`

Compare:
- instructions;
- message roles/order;
- tool definitions;
- function call/output representation;
- images;
- streaming fields;
- tool_choice;
- parallel tool flags;
- metadata/continuation identifiers.

## Deliverable

`docs/HERMES_RESPONSES_COMPATIBILITY.md`

Every difference must be classified COMPATIBLE, ADAPTABLE or BLOCKING.

---

# Gate G2 — Existing parser compatibility

Feed the Hermes fixture into the existing upstream Responses parser without launching a browser.

Assert:
- effective instructions are preserved;
- message order is preserved;
- tool schemas are correct;
- tool outputs map correctly;
- images survive normalization;
- no Codex-only assumption changes semantic content.

## PASS

Existing parser creates a semantically correct normalized request from a real Hermes fixture.

---

# Gate G3 — Harness abstraction

Introduce a narrow outer-runtime boundary.

Suggested types:

```ts
interface HarnessAdapter {
  identifyTurn(request: unknown): HarnessTurnIdentity;
  extractEnvironment(request: unknown): HarnessEnvironment;
  extractCapabilities(request: unknown): HarnessCapabilities;
  validateRequest(request: unknown): ValidationResult;
}
```

Create:
- `src/harness/types.ts`
- `src/harness/adapter.ts`
- `src/harness/hermes/*`

If useful during migration, keep a temporary `CodexHarnessAdapter` solely for upstream regression comparison.

Do not change browser behavior.

## PASS

All baseline upstream tests remain green and harness-neutral code no longer depends directly on Codex authority extraction.

---

# Gate G4 — Hermes provider plugin

Build the Hermes-side provider integration using `codex_responses`.

The provider resolves:
- local base URL;
- runtime bearer token;
- model catalog;
- streaming/cancellation.

Avoid forking Hermes core unless a failing compatibility test proves the plugin surface is insufficient.

## PASS

Hermes discovers/selects the provider and successfully sends a request to the local daemon.

---

# Gate G5 — Browser-only E2E

Disable MCP/tool calls.

Prove:

```text
Hermes -> bridge -> ChatGPT Web -> bridge -> Hermes
```

Test:
- simple prompt;
- multi-turn context;
- developer/system instruction;
- long input;
- streaming;
- cancellation;
- image input if supported;
- model/effort selection.

## PASS

Hermes receives a correct final model response through its normal agent path.

---

# Gate G6 — Single real tool round

Enable Full Mode/MCP.

Use a safe deterministic tool fixture, preferably read-only.

Required flow:

```text
ChatGPT MCP
 -> broker
 -> Responses function_call
 -> Hermes
 -> native Hermes tool dispatcher
 -> function_call_output
 -> broker
 -> same ChatGPT browser turn
 -> final
```

Assert:
- same `session_id`;
- same `turn_id`;
- new `round_id`;
- stable browser execution;
- stable active turn capability;
- stable `call_id`;
- tool executes exactly once.

## PASS

One complete real Hermes tool loop succeeds.

---

# Gate G7 — Multi-round tool loop

Run at least:

```text
tool A -> result -> reasoning -> tool B -> result -> reasoning -> tool C -> final
```

Assert:
- logical turn identity does not change;
- round counter advances;
- capability remains bound to the same turn;
- no browser restart occurs without explicit context/turn transition;
- every result routes to the correct call.

---

# Gate G8 — Errors, retry and idempotency

Add machine-readable error taxonomy.

At minimum test:
- stale capability;
- unknown tool;
- duplicate MCP request;
- duplicate identical tool result;
- conflicting result replay;
- late tool result;
- browser timeout;
- browser crash;
- malformed MCP message;
- Hermes cancellation;
- unauthorized local request.

Suggested codes:
- `AUTHORITY_LOST`
- `STALE_TURN`
- `CAPABILITY_DENIED`
- `TOOL_NOT_ADVERTISED`
- `DUPLICATE_CALL`
- `CONFLICTING_REPLAY`
- `BROWSER_TIMEOUT`
- `BROWSER_PROTOCOL_ERROR`
- `MCP_PROTOCOL_ERROR`
- `PROVIDER_INTERRUPTED`

No ambiguous failure may be converted into success.

---

# Gate G9 — Parallel tools and approvals

Test multiple outstanding calls and human approval waits.

Requirements:
- no duplicate execution;
- no result crossing between call IDs;
- no early completion;
- approval-pending does not retire the active turn;
- cancellation safely drains/rejects remaining calls.

---

# Gate G10 — Context/session ownership

Implement `context_epoch`.

Hermes owns durable history and agent-level compaction.

Bridge owns browser representation only.

If Hermes rebases/compresses context enough to invalidate browser retention:
- increment `context_epoch`;
- start a safe new browser context.

Never write browser conversation back as authoritative Hermes history.

---

# Gate G11 — Memory and delegation

Use Hermes-native memory and delegation.

Do not port Codex-specific subagent architecture unless needed by a proven missing primitive.

If a delegated Hermes child also uses ChatGPT Web, give it an isolated logical session/browser execution.

---

# Gate G12 — Auxiliary isolation

Only after the main loop is stable.

Default V1 auxiliary mode: disabled/minimal.

If ChatGPT Web is used for auxiliary work:
- `purpose=auxiliary`;
- separate browser execution;
- MCP off by default;
- no main-turn capability reuse;
- no contamination of main session state.

No auxiliary LLM may become an intermediary planner in the main path.

---

# Gate G13 — Launcher

Build a launcher that owns:
- bridge daemon lifecycle;
- Electron/browser runtime;
- Hermes provider connection/config;
- MCP runtime;
- runtime auth token;
- health/diagnostics.

Commands/UI actions:
- Start
- Stop
- Restart
- Login ChatGPT
- Health Check
- Smoke Test
- Diagnostics
- Logs

Headless runtime must work before GUI packaging.

---

# Gate G14 — Windows packaging

Primary target: Windows 10/11 x64.

Package a local application that can:
- start required processes;
- preserve the ChatGPT profile;
- display connection status;
- expose diagnostics;
- stop cleanly.

Keep UI decoupled from core runtime.

---

# Gate G15 — Security hardening

Audit:
- loopback binding;
- runtime bearer auth;
- Electron profile permissions;
- MCP capability lifecycle;
- trusted metadata schema validation;
- tool allowlist enforcement;
- stale turn rejection;
- prompt-injection boundary;
- diagnostics/log redaction;
- cancellation cleanup;
- dependency vulnerabilities.

Fail closed.

---

# Gate G16 — Release validation

Required release matrix:

- BUILD
- LINT
- TYPECHECK
- UNIT
- PARSER
- PROTOCOL
- BROWSER SMOKE
- HERMES BROWSER-ONLY E2E
- HERMES TOOL E2E
- MULTI-ROUND E2E
- CANCELLATION
- STALE CAPABILITY
- REPLAY
- PARALLEL TOOL
- APPROVAL
- CONTEXT EPOCH
- DELEGATION
- PACKAGE SMOKE
- DEPENDENCY AUDIT

Do not infer authenticated ChatGPT/MCP success from CI-only tests.

---

# Source strategy

## KEEP / PATCH_MINIMAL

Prefer to retain:
- browser worker;
- ChatGPT DOM/session code;
- Electron persistent profile;
- Responses parser;
- SSE/AdapterEvent conversion;
- MCP transport;
- model/effort selection;
- attachment handling.

## KEEP + GENERICIZE

Prefer minimal behavioral change:
- turn broker;
- turn execution;
- completion fencing;
- outstanding tool bookkeeping;
- cancellation;
- browser/task ownership;
- trace/usage state.

## MODIFY

Expected:
- ChatGPT adapter entry;
- prompt compiler;
- server/config;
- diagnostics;
- launcher.

## REWRITE

Expected:
- Codex identity extraction;
- Codex environment extraction;
- Codex setup/config integration;
- Hermes provider plugin;
- trusted runtime-auth metadata.

## REMOVE/DEFER

Expected:
- native Codex passthrough;
- Codex configuration patcher;
- Codex-specific installer;
- Codex-specific subagent/rollout assumptions.

---

# Commit discipline

Recommended change sequence:

1. baseline import
2. upstream reuse map
3. Hermes/Codex request fixtures
4. parser compatibility
5. harness abstraction
6. Hermes identity/environment adapter
7. Hermes provider plugin
8. browser-only E2E
9. single MCP tool integration
10. multi-round continuation
11. cancellation/replay protection
12. parallel tools/approval
13. context epoch
14. memory/delegation
15. auxiliary isolation
16. diagnostics
17. launcher
18. Windows package
19. security hardening
20. release validation

Do not combine many gates into one opaque commit.

---

# Required final deliverables

The finished repository must include:

- `README.md`
- `AGENTS.md`
- `docs/ARCHITECTURE.md`
- `docs/HERMES_CONTRACT.md`
- `docs/PROTOCOL.md`
- `docs/SECURITY_MODEL.md`
- `docs/UPSTREAM_REUSE_MAP.md`
- `docs/UPSTREAM_SYNC.md`
- `docs/TEST_PLAN.md`
- `docs/RELEASE_VALIDATION.md`
- `docs/TROUBLESHOOTING.md`
- `docs/DEVELOPMENT.md`
- exact upstream revision metadata;
- automated tests;
- redacted diagnostic bundle;
- Windows local launcher/package.

---

# Completion definition

MVP is reached when Hermes can:
1. discover/select the local provider;
2. send a `codex_responses` request;
3. receive streaming/final output from ChatGPT Web;
4. accept one real ChatGPT-requested Hermes tool call;
5. execute it through Hermes native tool runtime;
6. return the result to the same ChatGPT browser turn;
7. receive final answer;
8. cancel safely;
9. reject stale turn capabilities.

V1 additionally requires multi-round tools, parallel calls, approval handling, context epoch, basic delegation, diagnostics, security hardening and Windows packaging.

If implementation reality contradicts this plan, preserve working code, record the contradiction, create a minimal reproduction, and make the smallest testable architecture change. Do not silently redesign the system.
