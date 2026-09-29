# Agent Task — Transport-Neutral Remote Device Core

Status: READY_FOR_AGENT
Owner: Christopher Charman
Scope: Christopher-Charman/DesktopCommanderMCP

## Objective

Extract the useful remote-device reliability/execution machinery from the upstream Desktop Commander Remote implementation into transport-neutral interfaces so this fork can support an owned product-independent ingress without depending on mcp.desktopcommander.app or Supabase.

## Preserve

- `DesktopCommanderIntegration` stdio MCP supervision and verified-execution readiness.
- bounded restart/backoff and local child recovery.
- local duplicate-delivery suppression before execution.
- durable claim/result semantics for cross-process/restart safety.
- explicit heartbeat/readiness/online-offline semantics.
- atomic credential/session persistence patterns where a credentialed adapter needs them.
- graceful shutdown and bounded recovery.

## Replace / isolate behind adapters

- `mcp.desktopcommander.app`.
- vendor OAuth device flow.
- Supabase Auth and Realtime.
- hosted `mcp_devices` / `mcp_remote_calls` storage.
- vendor telemetry/capture coupling.

## Target architecture

```text
owned ingress
    -> authenticated request
    -> transport-neutral task/claim/result contract
    -> remote-device dispatcher
    -> MCP executor adapter
        -> existing PowerPC local MCP
        -> Desktop Commander MCP
        -> future Evenio/runtime-scoped MCPs
```

## Required first slice

Create explicit transport-neutral contracts for:

1. task envelope / target runtime identity / authority ceiling;
2. claim/idempotency;
3. terminal result receipt;
4. device readiness + heartbeat;
5. executor readiness/liveness;
6. transport adapter interface;
7. MCP executor interface.

Refactor current Supabase/hosted behavior behind an adapter without changing its behavior. Do **not** remove the current hosted adapter yet.

## Constraints

- No production PowerPC mutation in this task.
- No new generic shell capability.
- No widening of local MCP authority.
- Do not make Desktop Commander itself the architecture.
- Preserve exact-once/duplicate-suppression behavior and fail closed on ambiguous ownership.
- Keep the core usable with multiple runtime-scoped MCP executors.
- Avoid new dependencies where straightforward local code is sufficient.

## Acceptance

- transport-neutral core compiles independently of Supabase-specific classes;
- current hosted remote-device adapter still builds/tests;
- a fake/in-memory adapter can drive one bounded tool call through claim -> execute -> result;
- duplicate delivery does not execute twice;
- executor loss marks readiness false and bounded recovery restores it;
- tests cover claim, duplicate, result, shutdown, and recovery;
- no secrets or hosted credentials added to source.

This is a bounded architecture-extraction task. Do not redesign unrelated Desktop Commander features.
