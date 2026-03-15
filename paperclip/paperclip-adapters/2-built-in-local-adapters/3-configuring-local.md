---
source_course: "paperclip-adapters"
source_lesson: "paperclip-adapters-configuring-local"
---

# Configuring Local Adapters

## Introduction

All four local adapters in Paperclip — claude_local, codex_local, gemini_local, and opencode_local — share common configuration patterns. Once you understand how API keys, model selection, workspace paths, and monitoring work for one adapter, you can configure any of them. This lesson consolidates those shared patterns.

## Key Concepts

- **secret_ref Pattern**: The standard way to reference API keys stored in Paperclip's secret manager across all adapters
- **Model Selection**: How each adapter resolves model identifiers, from static lists to dynamic discovery
- **Workspace Passing**: The `cwd` configuration field that tells the adapter where the agent should operate
- **Run Monitoring**: How Paperclip tracks execution time, token usage, and costs across all adapter types

## Real World Context

A platform engineer is setting up Paperclip for a team of 20 developers. Each developer may use a different adapter, but the engineer wants consistent security practices (no plaintext API keys), consistent monitoring (usage tracking for all adapters), and consistent workspace isolation. By understanding the shared patterns, the engineer can write a single configuration template that works across all adapter types.

## Deep Dive

### API Key Management

Every local adapter needs an API key for its respective provider. The `secret_ref` pattern is consistent across all of them:

```json
{
  "env": {
    "ANTHROPIC_API_KEY": {
      "type": "secret_ref",
      "secretId": "anthropic-key-prod",
      "version": "latest"
    }
  }
}
```

For Codex, you would change the variable name to `OPENAI_API_KEY`. For Gemini, it becomes `GOOGLE_API_KEY`. The structure remains identical — only the environment variable name and secret ID change.

Paperclip resolves `secret_ref` values at runtime, just before calling the adapter's `execute()` function. The actual key never appears in configuration files, logs, or exports.

### Model Selection

Model selection differs slightly per adapter, but the configuration field is always `model`:

```json
{
  "adapterConfig": {
    "model": "claude-sonnet-4-20250514"
  }
}
```

For `opencode_local`, the model field includes the provider prefix:

```json
{
  "adapterConfig": {
    "model": "anthropic/claude-sonnet-4-20250514"
  }
}
```

Some adapters support dynamic model discovery. The Codex adapter queries the OpenAI API for available models and merges them with its built-in list. The Gemini adapter has a static list. The OpenCode adapter combines models from all configured providers.

### Workspace and Context Passing

The `cwd` field is the workspace directory. All local adapters pass this as the working directory when spawning the CLI process:

```json
{
  "adapterConfig": {
    "cwd": "/home/dev/projects/my-app",
    "promptTemplate": "You are {{agent.name}}. Workspace: {{cwd}}. Task: {{task}}"
  }
}
```

The workspace must be an absolute path. The adapter also passes Paperclip context through environment variables constructed by `buildPaperclipEnv()`. These include the run ID, task ID, and agent metadata.

### Monitoring and Transcripts

After each run, Paperclip captures execution data from the adapter's result:

```json
{
  "sessionId": "sess-abc123",
  "usage": {
    "inputTokens": 1250,
    "outputTokens": 3400,
    "totalTokens": 4650,
    "costUsd": 0.0234
  },
  "summary": "Refactored auth module into 3 files",
  "errors": []
}
```

This data structure is identical across all adapters. Paperclip aggregates it to show per-agent costs, per-run usage, and historical trends in the dashboard. The transcript viewer renders the parsed stdout so you can review exactly what the agent did during the run.

## Common Pitfalls

- **Using the wrong environment variable name for the API key**: Each provider expects a specific variable name. `ANTHROPIC_API_KEY` for Claude, `OPENAI_API_KEY` for Codex, `GOOGLE_API_KEY` for Gemini.
- **Mixing up static and dynamic model lists**: Not all adapters support dynamic discovery. Check the adapter's documentation to understand which models are available.
- **Forgetting that cwd must be absolute**: Relative paths resolve from Paperclip's process directory, leading to "workspace not found" errors.

## Best Practices

- **Create one secret per provider**: Store each provider's API key as a separate secret in Paperclip's secret manager with a descriptive ID like `anthropic-key-prod`.
- **Use prompt templates consistently**: Always use `renderTemplate()` placeholders rather than hardcoding values. This ensures prompts update automatically when agent or company metadata changes.
- **Monitor costs across adapters**: Use Paperclip's dashboard to compare per-adapter costs and identify which runtimes are most cost-effective for your workloads.

## Summary

- All local adapters use the `secret_ref` pattern for API key management
- Model selection uses the `model` field, with `provider/model` format for opencode_local
- The `cwd` field must be an absolute path to the workspace directory
- Paperclip captures usage metrics, costs, and transcripts uniformly across all adapter types
- Consistent patterns mean learning one adapter makes configuring the others straightforward

## Code Examples

**The secret_ref pattern is consistent across all local adapters — only the environment variable name and secret ID change per provider**

```json
{
  "env": {
    "ANTHROPIC_API_KEY": {
      "type": "secret_ref",
      "secretId": "anthropic-key-prod",
      "version": "latest"
    }
  }
}
```


## Resources

- [Paperclip Adapter Configuration Reference](https://paperclip.ing/docs/adapters/configuration) — Complete configuration reference for all built-in Paperclip adapters
- [Paperclip Run Monitoring](https://paperclip.ing/docs/monitoring) — Guide to monitoring agent runs, usage metrics, and costs in Paperclip

---

> 📘 *This lesson is part of the [Paperclip Adapters & Agent Runtimes](https://stanza.dev/courses/paperclip-adapters) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*