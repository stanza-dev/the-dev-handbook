---
source_course: "paperclip-fundamentals"
source_lesson: "paperclip-fundamentals-adapters-overview"
---

# Adapters at a Glance

## Introduction

Adapters are the bridge between Paperclip's control plane and the execution runtimes where agents actually do work. Paperclip ships with seven built-in adapters, each connecting to a different AI runtime or execution model. This adapter architecture is what makes Paperclip runtime-agnostic.

## Key Concepts

- **Adapter**: A plugin that translates Paperclip's execution requests into runtime-specific commands
- **Runtime Agnosticism**: The ability to swap agent runtimes without changing organizational structure or governance
- **Built-in Adapters**: Seven adapters ship with Paperclip out of the box
- **X-Paperclip-Run-Id**: A header passed through adapters to maintain audit trails across the execution boundary

## Real World Context

Consider a company that employs both full-time staff and contractors. Full-time staff work in the company's office (local adapters), while contractors work remotely via their own tools (HTTP adapter). A staffing agency might provide workers through their own platform (OpenClaw adapter). The company's management structure and policies are the same regardless of how each worker operates. Adapters provide this same flexibility for AI agents.

## Deep Dive

Paperclip's seven built-in adapters cover the most common AI runtimes and execution patterns:

```bash
# Built-in adapters:
# 1. claude_local   — Anthropic Claude via local CLI
# 2. codex_local    — OpenAI Codex via local CLI
# 3. gemini_local   — Google Gemini via local CLI
# 4. opencode_local — OpenCode via local CLI
# 5. openclaw       — OpenClaw agent runtime
# 6. process        — Generic process spawner
# 7. http           — HTTP-based agent endpoint
```

The first four adapters — claude_local, codex_local, gemini_local, and opencode_local — connect to AI model CLIs running on the same machine as Paperclip. They spawn a local process, pass the task context, and capture the output. These are the simplest to set up because they require no network configuration.

The **openclaw** adapter connects to an OpenClaw agent runtime, which provides multi-channel messaging capabilities. If your agent needs to communicate through chat platforms like Slack or Discord, this adapter bridges Paperclip's work management to OpenClaw's messaging infrastructure.

The **process** adapter is a generic spawner. It can run any executable — a Python script, a shell command, a compiled binary. This is useful for custom agent implementations that do not fit the model CLI pattern:

```bash
# Using the process adapter to run a custom script
# The adapter spawns the process and passes context via env vars
# PAPERCLIP_AGENT_ID, PAPERCLIP_RUN_ID, etc. are available
# The process reads these vars and calls back to the API
```

The process adapter gives you maximum flexibility. Your agent can be written in any language, use any tools, and follow any execution pattern — as long as it reads the Paperclip environment variables and calls back to the API when needed.

The **http** adapter sends execution requests to a remote HTTP endpoint. This is for agents running on separate servers or cloud functions:

```bash
# HTTP adapter sends a POST to the agent's endpoint
# POST https://my-agent.example.com/execute
# Body: { agent context, issue details, session state }
# Headers: X-Paperclip-Run-Id for audit tracing
```

The HTTP adapter enables distributed architectures where agents run in different environments — different servers, cloud regions, or even different organizations. The X-Paperclip-Run-Id header is passed through to maintain the audit trail across network boundaries.

Runtime agnosticism is the key benefit. You can have a team of agents where the CEO uses Claude, managers use Gemini, and ICs use Codex — all coordinated by the same Paperclip instance with the same org chart, budgets, and governance rules. Swapping an agent from Claude to Gemini is a configuration change, not a restructuring.

## Common Pitfalls

1. **Choosing an adapter before understanding the runtime** — Each adapter expects a specific runtime to be available. Verify the runtime is installed and configured before assigning an agent to use it.
2. **Using HTTP adapter for local agents** — If the agent runs on the same machine, use a local adapter for simplicity. The HTTP adapter adds network overhead unnecessarily.

## Best Practices

1. **Start with local adapters** — They are the easiest to set up and debug. Move to HTTP adapters only when you need distributed execution.
2. **Match adapters to agent strengths** — Different models excel at different tasks. Use Claude for complex reasoning, Codex for code generation, and Gemini for multimodal tasks.

## Summary

- Seven built-in adapters: claude_local, codex_local, gemini_local, opencode_local, openclaw, process, http
- Local adapters spawn CLI processes on the same machine as Paperclip
- The process adapter runs any executable for maximum flexibility
- The HTTP adapter enables distributed agent architectures
- Runtime agnosticism lets you mix models without changing org structure or governance
- X-Paperclip-Run-Id header maintains audit trails across adapter boundaries

## Code Examples

**List of all seven built-in Paperclip adapters**

```bash
# 7 built-in adapters:
# claude_local, codex_local, gemini_local,
# opencode_local, openclaw, process, http
```


## Resources

- [Paperclip Adapters Documentation](https://paperclip.ing/docs) — Official documentation on Paperclip adapters and runtime configuration
- [Paperclip GitHub Repository](https://github.com/paperclipai/paperclip) — Source code including adapter implementations

---

> 📘 *This lesson is part of the [Paperclip Fundamentals](https://stanza.dev/courses/paperclip-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*