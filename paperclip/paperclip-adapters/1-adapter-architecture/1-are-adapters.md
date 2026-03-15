---
source_course: "paperclip-adapters"
source_lesson: "paperclip-adapters-what-are-adapters"
---

# What Are Adapters?

## Introduction

Paperclip adapters are the connective tissue between Paperclip's orchestration engine and the AI agent runtimes that actually execute work. Every time Paperclip needs an agent to perform a task, it delegates to an adapter — and the adapter handles all the details of launching, communicating with, and capturing results from that specific runtime.

## Key Concepts

- **Adapter**: A pluggable module that knows how to invoke a specific agent runtime and capture its output
- **BYOA (Bring Your Own Agent)**: Paperclip's philosophy that any runtime capable of receiving heartbeats qualifies as an agent — you are never locked into a single provider
- **Runtime Agnosticism**: Paperclip does not care whether your agent is Claude, Codex, Gemini, a shell script, or a remote HTTP service — adapters abstract the differences away
- **Adapter Metadata**: Each adapter declares a type key, human-readable label, list of supported models, and documentation link
- **Built-in Adapters**: Paperclip ships with seven adapters: `claude_local`, `codex_local`, `gemini_local`, `opencode_local`, `openclaw`, `process`, and `http`

## Real World Context

Imagine a team where one developer prefers Claude Code for its deep reasoning, another uses Codex for its speed, and a third has a custom Python script that handles specialized data transformations. Without adapters, Paperclip would need to hardcode support for each runtime. With adapters, Paperclip treats every runtime identically at the orchestration level. The adapter handles the translation between Paperclip's unified interface and the runtime's specific API or CLI.

## Deep Dive

At its core, an adapter is a TypeScript module that exports metadata and an `execute()` function. The metadata tells Paperclip what the adapter is called, which models it supports, and where to find documentation. The `execute()` function does the actual work of invoking the runtime.

Here is what adapter metadata looks like in a typical adapter's root `index.ts`:

```typescript
import type { AdapterMetadata } from "@paperclipai/adapter-core";

export const metadata: AdapterMetadata = {
  type: "claude_local",
  label: "Claude Code (Local)",
  models: ["claude-sonnet-4-20250514", "claude-opus-4-20250514"],
  agentConfigurationDoc: "https://paperclip.ing/docs/adapters/claude-local"
};
```

The `type` field is the unique key that Paperclip uses to look up this adapter when an agent's configuration specifies `adapterType: "claude_local"`. The `label` is displayed in the UI. The `models` array tells the configuration form which model options to present. And `agentConfigurationDoc` links to the relevant documentation page.

Paperclip currently ships with seven built-in adapters, each targeting a different runtime:

```typescript
// All built-in adapter type keys
const ADAPTER_TYPES = [
  "claude_local",    // Claude Code CLI
  "codex_local",     // OpenAI Codex CLI
  "gemini_local",    // Google Gemini CLI
  "opencode_local",  // Multi-provider OpenCode CLI
  "openclaw",        // OpenClaw webhook-based agents
  "process",         // Arbitrary shell commands
  "http"             // HTTP webhook to external services
] as const;
```

Each of these type keys maps to a fully implemented adapter module with execution logic, UI components, and CLI formatting. The BYOA philosophy means you can also build your own custom adapter for any runtime not covered by the built-ins.

The key insight is that Paperclip's orchestration layer never interacts with agent runtimes directly. It always goes through an adapter. This indirection is what makes Paperclip runtime-agnostic — the orchestrator speaks one language (the adapter interface), and each adapter translates that into whatever the runtime expects.

## Common Pitfalls

- **Assuming adapters are only for AI models**: The `process` and `http` adapters demonstrate that any executable process or HTTP endpoint can be an agent. Adapters are not limited to LLM runtimes.
- **Confusing adapter type with model name**: The adapter type (e.g., `claude_local`) identifies which adapter module to use. The model (e.g., `claude-sonnet-4-20250514`) is a configuration parameter within that adapter.
- **Thinking BYOA means no structure**: BYOA does not mean anything goes. Every adapter must implement the same interface — `execute()` function, metadata, and module structure.

## Best Practices

- **Choose adapters based on your deployment model**: Use local adapters (`claude_local`, `codex_local`) when agents run on the same machine as Paperclip. Use `http` when agents are deployed as remote services.
- **Start with built-in adapters**: Paperclip's seven built-in adapters cover the most common runtimes. Only build a custom adapter when none of the built-ins fit.
- **Keep adapter metadata accurate**: The `models` array and `agentConfigurationDoc` link are surfaced in the UI. Outdated metadata confuses users configuring agents.

## Summary

- Adapters are pluggable modules that bridge Paperclip's orchestration engine and agent runtimes
- The BYOA philosophy means any runtime that can receive heartbeats qualifies as a Paperclip agent
- Each adapter declares metadata (type key, label, models, docs) and implements an `execute()` function
- Paperclip ships with seven built-in adapters: claude_local, codex_local, gemini_local, opencode_local, openclaw, process, and http
- Runtime agnosticism is achieved through the adapter interface — the orchestrator never talks to runtimes directly

## Code Examples

**Adapter metadata declares the type key, display label, supported models, and a link to configuration documentation**

```typescript
import type { AdapterMetadata } from "@paperclipai/adapter-core";

export const metadata: AdapterMetadata = {
  type: "claude_local",
  label: "Claude Code (Local)",
  models: ["claude-sonnet-4-20250514", "claude-opus-4-20250514"],
  agentConfigurationDoc: "https://paperclip.ing/docs/adapters/claude-local"
};
```


## Resources

- [Paperclip Adapters Documentation](https://paperclip.ing/docs/adapters) — Official documentation for Paperclip adapter architecture and configuration
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Source code for Paperclip including all built-in adapter implementations

---

> 📘 *This lesson is part of the [Paperclip Adapters & Agent Runtimes](https://stanza.dev/courses/paperclip-adapters) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*