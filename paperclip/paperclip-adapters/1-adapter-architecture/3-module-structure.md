---
source_course: "paperclip-adapters"
source_lesson: "paperclip-adapters-module-structure"
---

# Adapter Module Structure

## Introduction

Every Paperclip adapter is organized into three distinct modules: server, UI, and CLI. This separation ensures that execution logic, visual rendering, and terminal formatting are decoupled and can evolve independently. Understanding this structure is the foundation for configuring, debugging, and building adapters.

## Key Concepts

- **Server Module**: Contains `execute.ts` for core execution logic and `test.ts` for environment validation
- **UI Module**: Contains `parse-stdout.ts` for transcript rendering in the run viewer and `build-config.ts` for form generation
- **CLI Module**: Contains `format-event.ts` for terminal output formatting in watch mode
- **Module Registration**: Each module is registered in a separate registry (server, UI, CLI) so Paperclip loads only what is needed in each context
- **Module Discovery**: Paperclip discovers adapters by scanning registered modules at startup

## Real World Context

Consider a team where backend engineers maintain the adapter's execution logic, frontend engineers build the configuration forms, and DevOps engineers care about terminal output during CI runs. The three-module split means each team can work on their part without stepping on each other's code. A change to the CLI formatter never risks breaking the server-side execution logic.

## Deep Dive

An adapter's directory structure follows a consistent pattern across all built-in adapters:

```typescript
// Adapter directory layout
// adapters/claude-local/
//   src/
//     index.ts          — Root metadata (type, label, models, docs)
//     server/
//       execute.ts      — Core execution logic
//       test.ts         — Environment validation
//     ui/
//       parse-stdout.ts — Transcript rendering for run viewer
//       build-config.ts — Transform form input to adapter config JSON
//       config-fields.tsx — React form component for configuration
//     cli/
//       format-event.ts — Terminal pretty-printing for watch mode
```

The root `index.ts` exports the adapter metadata we saw in the previous lesson. Let us look at each module in detail.

### Server Module

The server module is where the action happens. The `execute.ts` file contains the core logic that spawns processes, makes HTTP requests, or otherwise communicates with the agent runtime:

```typescript
// server/execute.ts — simplified structure
import { AdapterExecutionContext, AdapterExecutionResult } from "@paperclipai/adapter-core";

export async function execute(ctx: AdapterExecutionContext): Promise<AdapterExecutionResult> {
  // 1. Read configuration
  // 2. Build environment
  // 3. Spawn process or make HTTP request
  // 4. Parse and return results
}
```

The `test.ts` file validates that the runtime environment is correctly set up. It returns diagnostic messages at three severity levels — error, warn, and info:

```typescript
// server/test.ts — environment validation
export async function test(): Promise<TestResult[]> {
  const results: TestResult[] = [];
  // Check if CLI is installed
  // Check if API key is configured
  // Check if workspace exists
  return results;
}
```

This validation runs when users click "Test" in the agent configuration UI, helping them diagnose setup issues before running agents.

### UI Module

The UI module handles everything visible in Paperclip's web interface. The `parse-stdout.ts` file converts raw process output into structured transcript entries that the run viewer can render:

```typescript
// ui/parse-stdout.ts — transcript rendering
export function parseStdout(stdout: string): TranscriptEntry[] {
  // Parse raw output into structured entries
  // Each entry has a type (text, code, tool_use, etc.)
  return entries;
}
```

The `build-config.ts` file transforms form input from the configuration UI into the JSON shape that the server module's `execute()` function expects.

### CLI Module

The CLI module formats adapter events for terminal output. When developers run Paperclip in watch mode, the `format-event.ts` file controls how events are pretty-printed:

```typescript
// cli/format-event.ts — terminal formatting
export function formatEvent(event: AdapterEvent): string {
  // Format event for terminal display
  // Uses colors, icons, and structured layout
  return formattedString;
}
```

### Registration

Each module is registered in its own registry. This means the server only loads server modules, the UI only loads UI modules, and the CLI only loads CLI modules:

```typescript
// Server registry
registerServerAdapter("claude_local", claudeLocalServer);

// UI registry
registerUIAdapter("claude_local", claudeLocalUI);

// CLI registry
registerCLIAdapter("claude_local", claudeLocalCLI);
```

This separation keeps bundle sizes small and prevents server-side code from accidentally importing React components or vice versa.

## Common Pitfalls

- **Editing the wrong module**: If transcript rendering is broken, the fix is in `ui/parse-stdout.ts`, not `server/execute.ts`. Always identify which module owns the behavior you need to change.
- **Forgetting to register all three modules**: An adapter that only registers its server module will work for execution but will not render transcripts or format CLI output.
- **Importing across module boundaries**: The server module must not import from the UI module or vice versa. Cross-module imports break the separation that keeps adapters maintainable.

## Best Practices

- **Keep the server module dependency-free**: The root metadata and server module should not depend on React, CSS frameworks, or CLI libraries. Only import from `@paperclipai/adapter-core` and `@paperclipai/adapter-utils`.
- **Test the test module**: Run `test.ts` validation on a clean machine to verify it catches real setup issues, not just issues specific to your development environment.
- **Use the existing adapters as templates**: When building a custom adapter, copy the directory structure of a built-in adapter and modify from there.

## Summary

- Every adapter has three modules: server (execution), UI (rendering), and CLI (formatting)
- The server module contains execute.ts for core logic and test.ts for validation
- The UI module contains parse-stdout.ts for transcripts and build-config.ts for configuration forms
- The CLI module contains format-event.ts for terminal output
- Modules are registered separately so each context loads only what it needs
- Cross-module imports are prohibited to maintain clean separation

## Code Examples

**Each adapter registers its three modules in separate registries so Paperclip loads only the relevant code in each context**

```typescript
// Server registry
registerServerAdapter("claude_local", claudeLocalServer);

// UI registry
registerUIAdapter("claude_local", claudeLocalUI);

// CLI registry
registerCLIAdapter("claude_local", claudeLocalCLI);
```


## Resources

- [Paperclip Custom Adapters Guide](https://paperclip.ing/docs/adapters/custom) — Official guide for building custom Paperclip adapters including module structure and registration
- [Paperclip GitHub — Adapter Examples](https://github.com/paperclipai/paperclip) — Source code for all built-in adapters showing the three-module directory structure

---

> 📘 *This lesson is part of the [Paperclip Adapters & Agent Runtimes](https://stanza.dev/courses/paperclip-adapters) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*