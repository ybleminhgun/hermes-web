# Architecture

## Target topology

```text
User
  |
  v
Hermes AIAgent
  |  codex_responses
  v
Hermes Web local daemon
  |- Responses parser
  |- Harness adapter
  |- ChatGPT Web adapter
  |- Turn execution
  |- Capability broker
  |- MCP server
  |- Browser worker pool
  `----> ChatGPT Web
           |
           | MCP tool request
           v
        Turn broker
           |
           | Responses function_call
           v
        Hermes AIAgent
           |
           v
        Hermes native tool runtime
```

## Responsibility split

### Hermes owns
- durable conversation/session state;
- long-term memory;
- prompt/history assembly;
- agent orchestration;
- tool definitions;
- tool execution;
- approvals;
- delegation/subagents;
- agent-level context compression.

### Bridge owns
- local Responses endpoint;
- trusted runtime metadata validation;
- browser/session lifecycle;
- ChatGPT Web interaction;
- model/effort selection;
- streaming conversion;
- MCP reverse transport;
- current-turn capability enforcement;
- browser completion fencing;
- transport cancellation and diagnostics.

### ChatGPT Web owns
- main reasoning;
- tool selection among capabilities exposed for the active turn;
- final model response.

## Main request lifecycle

1. Hermes builds an effective model request and current tool schema.
2. Hermes provider sends a Codex-style Responses request to loopback.
3. Bridge authenticates the caller and validates trusted Hermes metadata.
4. Responses parser converts the request to the upstream normalized request representation.
5. Harness adapter binds session/turn/round/environment/capabilities.
6. Browser worker submits compiled context to ChatGPT Web.
7. Bridge streams reasoning/text events back to Hermes.
8. If no tool is requested, final response closes the logical turn.

## Tool lifecycle

1. ChatGPT invokes the configured MCP connector.
2. MCP request includes the active ephemeral turn capability.
3. Turn broker verifies capability, current turn, and advertised tool.
4. Bridge emits a Responses `function_call` to Hermes.
5. Hermes executes through its normal tool runtime.
6. Hermes sends `function_call_output` on the next round of the same logical turn.
7. Bridge resolves the matching outstanding tool call.
8. Same ChatGPT browser turn resumes.
9. Completion is committed only after browser and tool activity are settled.

## Identity model

Canonical identity: `session_id`, optional `task_id`, `turn_id`, `round_id`, `request_id`, optional `parent_session_id`, `purpose`, `context_epoch`.

`turn_id` remains constant across tool rounds. `round_id` increments per model invocation. `context_epoch` increments when durable context changes enough that browser retention becomes unsafe.

## Context model

Hermes history is authoritative. Browser retention is only a transport optimization. When `context_epoch` changes, the bridge may create a new Temporary Chat rather than continue stale browser state.

## Auxiliary work

Auxiliary operations are outside the main reasoning chain. If ChatGPT Web is later used for auxiliary work, use `purpose=auxiliary`, no main-turn capability token reuse, MCP disabled by default, and a separate browser execution identity.

## Failure model

Fail closed on unknown/stale turn, unadvertised tool, conflicting replay, uncertain completion, connector mismatch, runtime-auth failure, or provider-stickiness violation.
