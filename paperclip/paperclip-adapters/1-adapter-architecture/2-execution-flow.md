---
source_course: "paperclip-adapters"
source_lesson: "paperclip-adapters-execution-flow"
---

# The Execution Flow

## Introduction

Every time Paperclip asks an agent to perform a task, a precise four-step execution flow unfolds inside the adapter layer. Understanding this flow is essential for debugging adapter issues, writing custom adapters, and reasoning about how context reaches your agent runtime.

## Key Concepts

- **Execution Flow**: The four-step sequence — retrieve config, invoke execute, launch runtime, capture results
- **AdapterExecutionContext**: The context object passed to every adapter's `execute()` function, containing agent config, run ID, task ID, and template data
- **buildPaperclipEnv()**: A utility that constructs environment variables for injecting Paperclip context into the agent runtime
- **renderTemplate()**: A function that interpolates template variables (agent ID, company ID, run ID) into prompt strings
- **AdapterExecutionResult**: The structured return value containing session ID, usage metrics, summary, and errors

## Real World Context

When a heartbeat triggers a run for a Claude-based agent, Paperclip does not simply shell out to the Claude CLI and hope for the best. It follows a deterministic flow: read the agent's adapter configuration, build the execution context, call the adapter's `execute()` function, and parse the structured results. This predictability is what allows Paperclip to track costs, monitor usage, and retry failed runs across any adapter type.

## Deep Dive

The adapter execution flow has four distinct steps. Let us trace each one.

**Step 1: Retrieve the agent's adapter type and configuration.**

Paperclip reads the agent record to determine which adapter to use and what configuration to pass:

```typescript
// Paperclip resolves the adapter from the agent's config
const agent = await getAgent(agentId);
const adapterType = agent.adapterType;  // e.g., "claude_local"
const adapterConfig = agent.adapterConfig; // JSON configuration object
```

The `adapterType` field maps to a registered adapter module. The `adapterConfig` contains runtime-specific settings like model selection, API keys (via secret_ref), timeouts, and workspace paths.

**Step 2: Invoke the adapter's `execute()` function with context.**

Paperclip constructs an `AdapterExecutionContext` and passes it to the adapter:

```typescript
import { AdapterExecutionContext, AdapterExecutionResult } from "@paperclipai/adapter-core";

export async function execute(ctx: AdapterExecutionContext): Promise<AdapterExecutionResult> {
  const apiKey = asString(ctx.config.apiKey);
  const model = asString(ctx.config.model);
  const prompt = renderTemplate(ctx.template, {
    agentId: ctx.agent.id,
    companyId: ctx.company.id,
    runId: ctx.runId
  });
  // ... execution logic follows
}
```

The context includes the agent's full configuration, the current run and task identifiers, and the prompt template. The `renderTemplate()` function replaces placeholders like `{{agent.id}}` with actual values.

**Step 3: Adapter launches or communicates with the agent runtime.**

The adapter translates the context into whatever the runtime expects. For local CLI adapters, this means spawning a child process with the right environment:

```typescript
const paperclipEnv = buildPaperclipEnv(ctx.agent);
const env = {
  ...paperclipEnv,
  PAPERCLIP_RUN_ID: ctx.runId,
  PAPERCLIP_TASK_ID: ctx.taskId,
  PAPERCLIP_API_KEY: apiKey
};

const result = await runChildProcess({
  command: "claude",
  args: ["--model", model, "--print", prompt],
  env,
  cwd: ctx.config.cwd
});
```

The `buildPaperclipEnv()` utility constructs a set of environment variables that inject Paperclip context into the runtime. These include the run ID, task ID, and API key. The adapter then spawns the CLI process with these variables.

**Step 4: Capture results — stdout, usage metrics, cost data.**

After the runtime completes, the adapter parses its output into a structured `AdapterExecutionResult`:

```typescript
return {
  sessionId: parsedOutput.sessionId,
  usage: parsedOutput.usage,
  summary: parsedOutput.summary,
  errors: parsedOutput.errors,
  clearSession: false
};
```

The result includes session state for continuity across runs, usage metrics for billing, a summary for the run viewer, and any errors for debugging. Paperclip uses this structured data to update its own records and decide whether to retry.

## Common Pitfalls

- **Forgetting to call buildPaperclipEnv()**: Without the Paperclip environment variables, the runtime cannot report back to Paperclip or access context about the current run.
- **Not handling errors in the result**: Adapters must populate the `errors` array even when the runtime fails. Swallowing errors silently makes debugging nearly impossible.
- **Ignoring session state**: The `sessionId` and `clearSession` fields control conversation continuity. Mishandling them causes agents to lose context between runs.

## Best Practices

- **Always use renderTemplate() for prompts**: Hardcoding prompt strings bypasses Paperclip's template system and breaks when agent or company metadata changes.
- **Return structured usage data**: Even if your runtime does not track tokens, return zeroed usage metrics so Paperclip's cost tracking remains consistent.
- **Log the execution context during development**: Temporarily logging `ctx` helps verify that Paperclip is passing the correct configuration to your adapter.

## Summary

- The execution flow has four steps: retrieve config, invoke execute, launch runtime, capture results
- AdapterExecutionContext provides the adapter with everything it needs: config, IDs, and template data
- buildPaperclipEnv() injects Paperclip context into the runtime's environment variables
- renderTemplate() interpolates agent and run metadata into prompt strings
- AdapterExecutionResult returns session state, usage metrics, summary, and errors to Paperclip

## Code Examples

**The complete execute function pattern showing config reading, environment building, template rendering, process spawning, and result capturing**

```typescript
export async function execute(ctx: AdapterExecutionContext): Promise<AdapterExecutionResult> {
  const apiKey = asString(ctx.config.apiKey);
  const paperclipEnv = buildPaperclipEnv(ctx.agent);
  const env = { ...paperclipEnv, PAPERCLIP_RUN_ID: ctx.runId, PAPERCLIP_TASK_ID: ctx.taskId };
  const prompt = renderTemplate(ctx.template, { agentId: ctx.agent.id, companyId: ctx.company.id, runId: ctx.runId });
  const result = await runChildProcess({ command: "claude", args: ["--model", model, "--print", prompt], env, cwd: ctx.config.cwd });
  return { sessionId: result.sessionId, usage: result.usage, summary: result.summary, errors: result.errors, clearSession: false };
}
```


## Resources

- [Paperclip Adapter Execution Guide](https://paperclip.ing/docs/adapters/execution) — Official documentation on the adapter execution lifecycle and context object
- [Paperclip GitHub — Adapter Core](https://github.com/paperclipai/paperclip) — Source code for the adapter core types including AdapterExecutionContext and AdapterExecutionResult

---

> 📘 *This lesson is part of the [Paperclip Adapters & Agent Runtimes](https://stanza.dev/courses/paperclip-adapters) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*