---
source_course: "paperclip-adapters"
source_lesson: "paperclip-adapters-validation-ui-cli"
---

# Validation, UI & CLI Modules

## Introduction

The server execute function is the core of your adapter, but a production-ready adapter also needs validation, transcript rendering, configuration forms, and terminal formatting. These three supporting concerns live in the test, UI, and CLI modules. This lesson covers all of them.

## Key Concepts

- **test.ts**: Validates that the runtime environment is correctly set up, returning diagnostics at error, warn, and info severity levels
- **parse-stdout.ts**: Converts raw process or HTTP output into structured transcript entries for the run viewer
- **build-config.ts**: Transforms form input from the UI into the adapter configuration JSON that `execute()` expects
- **config-fields.tsx**: A React component that renders the configuration form in Paperclip's agent setup UI
- **format-event.ts**: Formats adapter events for terminal display during watch mode

## Real World Context

A developer installs your custom adapter and tries to configure it. Without `test.ts`, they would not know if their API key is valid or if the runtime is installed. Without `config-fields.tsx`, they would have to write raw JSON. Without `parse-stdout.ts`, they would see garbled text instead of a clean transcript. These modules transform a functional adapter into a polished one.

## Deep Dive

### Validation Module (test.ts)

The test module runs diagnostic checks and returns results at three severity levels:

```typescript
import type { TestResult } from "@paperclipai/adapter-core";

export async function test(): Promise<TestResult[]> {
  const results: TestResult[] = [];

  // Error level: blocks agent creation
  const serverReachable = await checkServer();
  if (!serverReachable) {
    results.push({
      level: "error",
      message: "Cannot reach vLLM server at configured URL"
    });
  }

  // Warn level: allows creation but flags concern
  const modelAvailable = await checkModel();
  if (!modelAvailable) {
    results.push({
      level: "warn",
      message: "Configured model not found; default model will be used"
    });
  }

  // Info level: informational only
  results.push({
    level: "info",
    message: `Server version: ${await getServerVersion()}`
  });

  return results;
}
```

The three severity levels serve different purposes. An `error` blocks agent creation — the user must fix the issue before proceeding. A `warn` allows creation but alerts the user to potential problems. An `info` provides useful context without indicating any issue.

### UI Module — Transcript Parsing (parse-stdout.ts)

The `parse-stdout.ts` file converts raw output into transcript entries that the run viewer renders:

```typescript
import type { TranscriptEntry } from "@paperclipai/adapter-core";

export function parseStdout(stdout: string): TranscriptEntry[] {
  const entries: TranscriptEntry[] = [];
  const lines = stdout.split("\n");

  for (const line of lines) {
    if (line.startsWith("[THINKING]")) {
      entries.push({ type: "thinking", content: line.slice(10) });
    } else if (line.startsWith("[CODE]")) {
      entries.push({ type: "code", content: line.slice(6), language: "typescript" });
    } else if (line.trim()) {
      entries.push({ type: "text", content: line });
    }
  }

  return entries;
}
```

Each entry has a `type` that tells the run viewer how to render it. Text entries get normal formatting, code entries get syntax highlighting, and thinking entries get a collapsed view.

### UI Module — Config Building (build-config.ts)

The `build-config.ts` file transforms form input into the adapter configuration JSON:

```typescript
export function buildConfig(formData: Record<string, unknown>): Record<string, unknown> {
  return {
    serverUrl: formData.serverUrl,
    model: formData.model,
    timeoutMs: Number(formData.timeoutMs) || 30000,
    env: {
      API_KEY: formData.apiKeySecret
        ? { type: "secret_ref", secretId: formData.apiKeySecret, version: "latest" }
        : undefined
    }
  };
}
```

This function bridges the gap between the UI form (which uses flat key-value pairs) and the adapter config (which has nested structures and typed fields).

### CLI Module (format-event.ts)

The CLI module formats events for terminal output during watch mode:

```typescript
import type { AdapterEvent } from "@paperclipai/adapter-core";

export function formatEvent(event: AdapterEvent): string {
  const timestamp = new Date(event.timestamp).toLocaleTimeString();
  const icon = event.type === "start" ? "▶" : event.type === "complete" ? "✓" : "✗";
  return `${icon} [${timestamp}] ${event.summary}`;
}
```

This function is called whenever the adapter emits an event during execution. The formatted string is printed directly to the terminal.

## Common Pitfalls

- **Only implementing error-level validation**: Warn and info levels provide valuable feedback. A test module that only returns errors misses the opportunity to help users optimize their configuration.
- **Parsing stdout too aggressively**: If your parser throws on unexpected output, the entire transcript fails to render. Always handle unknown output gracefully.
- **Forgetting to handle empty form fields in build-config**: Users may leave optional fields blank. Build-config must handle undefined values without producing invalid JSON.

## Best Practices

- **Test your validation on a clean machine**: Run `test.ts` on a machine without the runtime installed to verify it catches the missing dependency.
- **Match transcript entry types to the run viewer's capabilities**: Check Paperclip's documentation for all supported entry types (text, code, thinking, tool_use, error) before implementing your parser.
- **Keep format-event.ts simple**: Terminal formatting should be fast and side-effect-free. Complex formatting slows down watch mode.

## Summary

- The test module validates the runtime environment at three severity levels: error, warn, and info
- parse-stdout.ts converts raw output into structured transcript entries for the run viewer
- build-config.ts transforms UI form input into the nested adapter configuration JSON
- config-fields.tsx provides a React form component for the agent configuration UI
- format-event.ts formats adapter events for terminal display in watch mode

## Code Examples

**Validation module returning diagnostics at error and info severity levels — errors block agent creation while info provides context**

```typescript
export async function test(): Promise<TestResult[]> {
  const results: TestResult[] = [];
  const serverReachable = await checkServer();
  if (!serverReachable) {
    results.push({ level: "error", message: "Cannot reach server" });
  }
  results.push({ level: "info", message: `Version: ${await getServerVersion()}` });
  return results;
}
```


## Resources

- [Paperclip Adapter UI Components](https://paperclip.ing/docs/adapters/ui) — Guide to building UI components for custom Paperclip adapters
- [Paperclip Run Viewer Documentation](https://paperclip.ing/docs/run-viewer) — Documentation for transcript entry types and run viewer rendering

---

> 📘 *This lesson is part of the [Paperclip Adapters & Agent Runtimes](https://stanza.dev/courses/paperclip-adapters) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*