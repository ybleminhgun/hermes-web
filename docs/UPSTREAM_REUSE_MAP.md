# Upstream Reuse Map

Reference upstream: `miuuyy/codex-chatgpt-web`

This file is a Phase G0 template. Antigravity must update it with the exact upstream SHA and exact source paths after auditing the checked-out revision.

## KEEP / PATCH_MINIMAL

Expected:
- browser worker / ChatGPT DOM automation;
- ChatGPT session/profile ownership;
- Electron browser host;
- model/effort selection;
- attachment handling;
- response extraction/streaming;
- Responses parsing;
- AdapterEvent -> Responses/SSE translation;
- MCP transport.

## KEEP + GENERICIZE

Expected:
- turn broker;
- turn execution;
- completion fencing;
- outstanding tool-call bookkeeping;
- cancellation lifecycle;
- task/browser ownership keys;
- tracing/usage metadata.

## MODIFY

Expected:
- ChatGPT adapter entry point;
- prompt compiler;
- server routing;
- configuration;
- diagnostics;
- launcher integration.

## REWRITE

Expected:
- Codex environment extraction;
- Codex identity extraction;
- Codex setup/config patching;
- Hermes provider integration;
- trusted runtime metadata/auth.

## REMOVE OR DEFER

Expected:
- native Codex passthrough;
- Codex-specific installer/config mutation;
- Codex-specific subagent protocol;
- Codex-specific rollout recovery.

## Anti-pattern

Do not mass-rename legacy internal types before E2E works. Temporary legacy names are acceptable if semantics are correct and they reduce upstream diff.

## Required source-level audit table

| Upstream path | Status | Hermes replacement | Reason | Tests |
|---|---|---|---|---|
| TBD | TBD | TBD | TBD | TBD |
