---
source_course: "paperclip-adapters"
source_lesson: "paperclip-adapters-server-implementation"
---

# Server-Side Implementation

## Introduction

The server module is the heart of any custom adapter. Its `execute.ts` file contains the core logic that connects Paperclip's orchestration to your target runtime. This lesson walks through implementing a complete execute function, from reading configuration to handling session errors.

## Key Concepts

- **execute() Function**: The async function that receives an AdapterExecutionContext and returns an AdapterExecutionResult
- **Config Helpers**: Type-safe utilities like `asString()`, `asNumber()`, `asBoolean()` for reading configuration values
- **buildPaperclipEnv()**: Constructs environment variables that inject Paperclip context into the runtime
- **Session Management**: Retrieving existing sessions, updating state, and handling session errors with retry
- **Output Parsing**: Extracting usage metrics, costs, session updates, and errors from the runtime's output

## Real World Context

You are building a custom adapter for a team that runs a fine-tuned model on a local GPU server. The model exposes a REST API. Your execute function needs to read the server URL from config, send the prompt as an HTTP request, parse the response for token counts, and return structured results. If the session state is corrupted, it needs to clear the session and retry.

## Deep Dive

The execute function follows a predictable eight-step pattern. Let us implement each step:

```typescript
import { AdapterExecutionContext, AdapterExecutionResult } from "@paperclipai/adapter-core";
import { asString, asNumber } from "@paperclipai/adapter-utils/server-utils";
import { buildPaperclipEnv, renderTemplate } from "./utils";

export async function execute(ctx: AdapterExecutionContext): Promise<AdapterExecutionResult> {
  // Step 1: Read configuration safely
  const serverUrl = asString(ctx.config.serverUrl);
  const model = asString(ctx.config.model);
  const timeoutMs = asNumber(ctx.config.timeoutMs, 30000);
```

The `asString()` and `asNumber()` helpers validate and cast configuration values. The second argument to `asNumber()` provides a default value if the field is missing. These helpers throw clear error messages when required fields are absent.

```typescript
  // Step 2: Build environment
  const paperclipEnv = buildPaperclipEnv(ctx.agent);
  const env = {
    ...paperclipEnv,
    PAPERCLIP_RUN_ID: ctx.runId,
    PAPERCLIP_TASK_ID: ctx.taskId
  };
```

The `buildPaperclipEnv()` function creates a standard set of environment variables from the agent's metadata. You add the run and task IDs on top.

```typescript
  // Step 3: Resolve session state
  let sessionId = ctx.existingSessionId || null;
  let sessionState = sessionId ? await getSessionState(sessionId) : null;
```

Sessions allow agents to maintain context across runs. If an existing session ID is provided, the adapter retrieves its state. For the first run, there is no session.

```typescript
  // Step 4: Render prompt template
  const prompt = renderTemplate(ctx.template, {
    agentId: ctx.agent.id,
    companyId: ctx.company.id,
    runId: ctx.runId,
    session: sessionState
  });
```

The `renderTemplate()` function replaces `{{...}}` placeholders with actual values. The session state is passed so prompts can reference previous conversation context.

```typescript
  // Step 5: Make request to runtime
  try {
    const response = await fetch(`${serverUrl}/v1/completions`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ model, prompt, session: sessionState }),
      signal: AbortSignal.timeout(timeoutMs)
    });
    const output = await response.json();
```

This step is runtime-specific. For a local GPU server, it is an HTTP request. For a CLI tool, it would be `runChildProcess()`. The timeout ensures the adapter does not hang.

```typescript
    // Step 6: Parse output
    const usage = {
      inputTokens: output.usage?.prompt_tokens || 0,
      outputTokens: output.usage?.completion_tokens || 0,
      totalTokens: output.usage?.total_tokens || 0,
      costUsd: calculateCost(output.usage)
    };
```

Parsing extracts structured data from the runtime's response. Even if the runtime does not track tokens, return zeroed metrics for consistency.

```typescript
    // Step 7: Return result
    return {
      sessionId: output.sessionId || sessionId,
      usage,
      summary: output.text || "Completed",
      errors: [],
      clearSession: false
    };
  } catch (error) {
    // Step 8: Handle session errors
    if (isSessionError(error) && sessionId) {
      return {
        sessionId: null,
        usage: { inputTokens: 0, outputTokens: 0, totalTokens: 0, costUsd: 0 },
        summary: "Session error — cleared for retry",
        errors: [{ message: error.message, code: "SESSION_ERROR" }],
        clearSession: true
      };
    }
    throw error;
  }
}
```

The session error handler is critical. When a session becomes corrupted, setting `clearSession: true` tells Paperclip to discard the session state and start fresh on the next run. This prevents agents from getting stuck in a broken state.

## Common Pitfalls

- **Not using the config helper functions**: Accessing `ctx.config.serverUrl` directly gives you `unknown`. Always use `asString()`, `asNumber()`, etc. for type safety.
- **Ignoring session errors**: If the runtime reports a session error and you do not set `clearSession: true`, the next run will fail with the same corrupted state.
- **Throwing raw errors**: Always catch runtime errors and return them in the `errors` array. Unhandled exceptions crash the adapter without giving Paperclip usable diagnostic information.

## Best Practices

- **Return usage metrics even when zero**: Paperclip aggregates usage data across all adapters. Missing metrics show as gaps in the dashboard.
- **Test with invalid sessions**: Deliberately corrupt a session to verify your error handling works correctly before deploying.
- **Use AbortSignal.timeout() for all network requests**: This ensures your adapter cannot hang indefinitely waiting for a response.

## Summary

- The execute function follows eight steps: read config, build env, resolve session, render template, call runtime, parse output, return result, handle errors
- Config helpers (asString, asNumber) provide type-safe access to adapter configuration
- buildPaperclipEnv() creates standard environment variables from agent metadata
- Session management allows agents to maintain context across runs
- Setting clearSession to true recovers from corrupted session state

## Code Examples

**A complete custom adapter execute function showing config reading, environment building, HTTP request to runtime, and structured result return**

```typescript
export async function execute(ctx: AdapterExecutionContext): Promise<AdapterExecutionResult> {
  const serverUrl = asString(ctx.config.serverUrl);
  const model = asString(ctx.config.model);
  const paperclipEnv = buildPaperclipEnv(ctx.agent);
  const prompt = renderTemplate(ctx.template, { agentId: ctx.agent.id, runId: ctx.runId });

  const response = await fetch(`${serverUrl}/v1/completions`, {
    method: "POST",
    body: JSON.stringify({ model, prompt }),
    signal: AbortSignal.timeout(30000)
  });
  const output = await response.json();

  return {
    sessionId: output.sessionId,
    usage: { inputTokens: output.usage.prompt_tokens, outputTokens: output.usage.completion_tokens, totalTokens: output.usage.total_tokens, costUsd: 0 },
    summary: output.text,
    errors: [],
    clearSession: false
  };
}
```


## Resources

- [Paperclip Adapter Core API](https://paperclip.ing/docs/adapters/api) — API reference for AdapterExecutionContext, AdapterExecutionResult, and utility functions
- [Paperclip GitHub — Adapter Utils](https://github.com/paperclipai/paperclip) — Source code for adapter utility functions including asString, asNumber, buildPaperclipEnv, and renderTemplate

---

> 📘 *This lesson is part of the [Paperclip Adapters & Agent Runtimes](https://stanza.dev/courses/paperclip-adapters) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*