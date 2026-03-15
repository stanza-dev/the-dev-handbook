---
source_course: "paperclip-adapters"
source_lesson: "paperclip-adapters-claude-local"
---

# Claude Local Adapter

## Introduction

The Claude local adapter (`claude_local`) is one of Paperclip's most popular built-in adapters. It invokes the Claude Code CLI directly on your machine, giving your agents access to Claude's deep reasoning capabilities without leaving your local environment. This lesson covers how to configure it, pass context, and review run transcripts.

## Key Concepts

- **Adapter Type**: `claude_local` — the unique key identifying this adapter in Paperclip's registry
- **Claude Code CLI**: The command-line interface for Claude that the adapter spawns as a child process
- **secret_ref**: A secure reference to an API key stored in Paperclip's secret manager, avoiding plaintext credentials in configuration
- **Workspace**: The working directory passed to the Claude CLI via the `cwd` configuration option
- **Run Viewer Transcripts**: Parsed stdout from Claude displayed as structured conversation entries in Paperclip's UI

## Real World Context

A development team uses Paperclip to orchestrate multiple AI agents. Their primary coding agent uses Claude Sonnet for complex refactoring tasks. Instead of manually running Claude Code in a terminal, they configure the `claude_local` adapter so Paperclip can trigger Claude runs automatically on heartbeats, track usage, and display transcripts in the run viewer.

## Deep Dive

The `claude_local` adapter works by spawning the Claude Code CLI as a child process. Here is a typical agent configuration that uses this adapter:

```json
{
  "adapterType": "claude_local",
  "adapterConfig": {
    "cwd": "/absolute/path/to/workspace",
    "model": "claude-sonnet-4-20250514",
    "promptTemplate": "You are {{agent.name}} working for {{company.name}}. Your capabilities: {{capabilities}}",
    "timeoutSec": 300,
    "graceSec": 30,
    "maxTurnsPerRun": 10,
    "dangerouslySkipPermissions": false,
    "env": {
      "ANTHROPIC_API_KEY": {
        "type": "secret_ref",
        "secretId": "secret-abc123",
        "version": "latest"
      },
      "CUSTOM_VAR": "static-value"
    }
  }
}
```

Let us break down each field. The `cwd` field specifies the workspace directory where Claude will operate. This is the directory Claude sees when it executes file operations. The `model` field selects which Claude model to use.

The `promptTemplate` field uses Paperclip's template syntax. Placeholders like `{{agent.name}}` and `{{company.name}}` are replaced with actual values at runtime by the `renderTemplate()` function.

The `timeoutSec` and `graceSec` fields control execution time. If Claude does not complete within 300 seconds, Paperclip sends a graceful shutdown signal and waits 30 more seconds before force-killing the process. The `maxTurnsPerRun` limits how many conversation turns Claude can take in a single run.

The `env` block is where API keys live. Notice that `ANTHROPIC_API_KEY` uses a `secret_ref` instead of a plaintext value. This tells Paperclip to resolve the actual key from its secret manager at runtime:

```typescript
// How Paperclip resolves secret_ref values
const apiKey = await resolveSecret({
  secretId: "secret-abc123",
  version: "latest"
});
// apiKey is now the actual API key string
```

This pattern keeps credentials out of configuration files and version control. You can also set static environment variables like `CUSTOM_VAR` for values that are not sensitive.

When the adapter executes, it spawns the Claude CLI with all the configured options:

```typescript
const result = await runChildProcess({
  command: "claude",
  args: [
    "--model", config.model,
    "--print",
    renderedPrompt
  ],
  env: resolvedEnv,
  cwd: config.cwd,
  timeoutMs: config.timeoutSec * 1000
});
```

The `--print` flag tells Claude to output the response to stdout rather than entering interactive mode. The adapter captures this stdout and passes it to the UI module's `parse-stdout.ts`, which converts it into structured transcript entries visible in Paperclip's run viewer.

## Common Pitfalls

- **Using a relative path for cwd**: The workspace path must be absolute. A relative path resolves from Paperclip's process directory, not from where you expect.
- **Hardcoding API keys in adapterConfig**: Always use `secret_ref` for API keys. Hardcoded keys end up in configuration exports and audit logs.
- **Setting dangerouslySkipPermissions to true in production**: This flag bypasses Claude's permission system. Use it only during local development and never in production environments.

## Best Practices

- **Start with claude-sonnet for most tasks**: Sonnet offers a strong balance of capability and speed. Reserve Opus for tasks that genuinely require deeper reasoning.
- **Set reasonable timeouts**: A 300-second timeout works for most coding tasks. Increase it only for large-scale refactoring jobs.
- **Review transcripts after the first few runs**: Check the run viewer to verify that your prompt template produces the expected behavior before scaling up.

## Summary

- The `claude_local` adapter invokes Claude Code CLI as a local child process
- Configuration includes model selection, workspace path, prompt template, timeouts, and environment variables
- API keys should always use `secret_ref` to avoid plaintext credentials
- The adapter captures stdout and renders it as structured transcripts in the run viewer
- Timeouts and turn limits protect against runaway executions

## Code Examples

**Minimal claude_local adapter configuration showing model selection, workspace path, timeout, and secure API key reference**

```json
{
  "adapterType": "claude_local",
  "adapterConfig": {
    "cwd": "/absolute/path/to/workspace",
    "model": "claude-sonnet-4-20250514",
    "timeoutSec": 300,
    "env": {
      "ANTHROPIC_API_KEY": {
        "type": "secret_ref",
        "secretId": "secret-abc123",
        "version": "latest"
      }
    }
  }
}
```


## Resources

- [Paperclip Claude Adapter Documentation](https://paperclip.ing/docs/adapters/claude-local) — Official documentation for configuring the Claude local adapter in Paperclip
- [Paperclip Secrets Management](https://paperclip.ing/docs/secrets) — Guide to managing API keys and secrets with secret_ref in Paperclip

---

> 📘 *This lesson is part of the [Paperclip Adapters & Agent Runtimes](https://stanza.dev/courses/paperclip-adapters) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*