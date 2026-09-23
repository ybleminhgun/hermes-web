# Test Plan

## Status vocabulary

Every mandatory gate is exactly one of: PASS, FAIL, BLOCKED, NOT_TESTED.

Do not report PASS with skipped mandatory assertions.

## Protocol tests
- P001 basic Responses request
- P002 system/developer instructions
- P003 multi-message history
- P004 tool schema parsing
- P005 function_call_output parsing
- P006 image input
- P007 streaming
- P008 trusted Hermes metadata
- P009 unauthorized local request rejection

## Browser tests
- B001 persisted ChatGPT login
- B002 Temporary Chat creation
- B003 model selection
- B004 effort selection
- B005 prompt submission
- B006 logical response-turn detection
- B007 streaming extraction
- B008 cancellation
- B009 browser restart/recovery

## Tool tests
- T001 single MCP tool request
- T002 multi-round tool continuation
- T003 unknown tool rejected
- T004 stale capability rejected
- T005 duplicate request idempotency
- T006 duplicate result idempotency
- T007 conflicting replay rejected
- T008 tool failure returned correctly
- T009 parallel calls
- T010 approval wait
- T011 late tool request after completion rejected

## Identity tests
- I001 stable session binding
- I002 turn constant across tool rounds
- I003 round increments
- I004 request IDs unique
- I005 context epoch changes browser ownership
- I006 child/delegated session isolation

## Failure tests
- F001 browser crash
- F002 MCP disconnect
- F003 Hermes disconnect
- F004 cancellation race
- F005 response timeout
- F006 malformed tool result
- F007 conflicting trusted metadata
- F008 provider-stickiness violation

## Concurrency/race tests
Repeat enough iterations to expose races:
- completion vs late tool call;
- cancellation vs tool result;
- token retirement vs MCP request;
- browser failure vs result delivery;
- concurrent parallel tool results;
- session shutdown vs final browser completion.

Targets:
- 0 dual authority;
- 0 stale capability accepted;
- 0 duplicate side effects;
- 0 deadlocks.

## E2E milestones

### E2E-1 Browser-only
Hermes -> bridge -> ChatGPT Web -> Hermes final.

### E2E-2 Single tool
ChatGPT -> MCP -> broker -> Hermes native tool -> result -> same ChatGPT turn -> final.

### E2E-3 Multi-round
At least 3 sequential tool rounds under one logical turn.

### E2E-4 Cancellation
Abort from Hermes retires the browser turn and all tool authority.

### E2E-5 Context epoch
Hermes durable compaction/rebase starts a safe new browser context when required.
