---
source_course: "paperclip-adapters"
source_lesson: "paperclip-adapters-codex-gemini"
---

# Codex & Gemini Adapters

## Introduction

Beyond Claude, Paperclip ships with adapters for OpenAI Codex, Google Gemini, and the multi-provider OpenCode CLI. Each adapter follows the same architecture — metadata, execute function, three modules — but targets a different runtime. This lesson covers the unique characteristics of each and helps you choose between them.

## Key Concepts

- **codex_local**: Adapter that invokes the OpenAI Codex CLI for code generation and editing tasks
- **gemini_local**: Adapter that invokes the Google Gemini CLI for multimodal and reasoning tasks
- **opencode_local**: Adapter that supports multiple AI providers through a single CLI using a `provider/model` format
- **Model Discovery**: Each adapter discovers available models differently — some query an API, others merge local configs with provider defaults

## Real World Context

A company runs Paperclip with different agents for different tasks. Their code generation agent uses Codex for fast completions, their research agent uses Gemini for its large context window, and their polyglot agent uses OpenCode to switch between providers on a per-task basis. Paperclip's adapter system makes this multi-runtime setup seamless — each agent simply specifies a different `adapterType`.

## Deep Dive

### Codex Local Adapter

The `codex_local` adapter invokes OpenAI's Codex CLI. It follows the same spawn-and-capture pattern as the Claude adapter:

```typescript
// Codex adapter metadata
export const metadata: AdapterMetadata = {
  type: "codex_local",
  label: "Codex (Local)",
  models: ["codex-mini", "o4-mini", "o3"],
  agentConfigurationDoc: "https://paperclip.ing/docs/adapters/codex-local"
};
```

A key difference is model discovery. The Codex adapter merges locally configured models with options fetched from the OpenAI API, giving you access to both standard and fine-tuned models:

```typescript
// Codex model discovery merges local and remote models
const localModels = getLocalCodexModels();
const remoteModels = await fetchOpenAIModels(apiKey);
const allModels = [...new Set([...localModels, ...remoteModels])];
```

This approach ensures that custom or fine-tuned models deployed to your OpenAI account appear automatically in Paperclip's model selector.

### Gemini Local Adapter

The `gemini_local` adapter targets Google's Gemini CLI. Its configuration mirrors the pattern you have seen, with Gemini-specific model options:

```typescript
// Gemini adapter metadata
export const metadata: AdapterMetadata = {
  type: "gemini_local",
  label: "Gemini (Local)",
  models: ["gemini-2.5-pro", "gemini-2.5-flash"],
  agentConfigurationDoc: "https://paperclip.ing/docs/adapters/gemini-local"
};
```

The Gemini adapter uses a Google API key instead of an Anthropic or OpenAI key. The `secret_ref` pattern works identically — just reference a different secret ID.

### OpenCode Local Adapter

The `opencode_local` adapter is unique because it supports multiple providers through a single CLI. Instead of specifying just a model name, you use a `provider/model` format:

```typescript
// OpenCode adapter supports provider/model format
export const metadata: AdapterMetadata = {
  type: "opencode_local",
  label: "OpenCode (Local)",
  models: [
    "anthropic/claude-sonnet-4-20250514",
    "openai/gpt-4o",
    "google/gemini-2.5-pro"
  ],
  agentConfigurationDoc: "https://paperclip.ing/docs/adapters/opencode-local"
};
```

The `provider/model` format tells the OpenCode CLI which provider to route the request to. This makes `opencode_local` the most flexible adapter — a single configuration can target any supported provider without switching adapter types.

### Choosing Between Adapters

Here is a quick decision framework:

```typescript
// Decision framework (pseudocode)
if (needsDeepReasoning) use "claude_local";
if (needsFastCodeGen) use "codex_local";
if (needsMultimodal) use "gemini_local";
if (needsProviderFlexibility) use "opencode_local";
```

In practice, the choice often comes down to which provider's models your team already has API access to and which runtime's CLI is already installed.

## Common Pitfalls

- **Using the wrong API key type**: Each adapter expects a specific provider's API key. Passing an Anthropic key to the Codex adapter will fail silently or produce authentication errors.
- **Forgetting the provider prefix with opencode_local**: The model field must use `provider/model` format. Specifying just the model name will cause a lookup failure.
- **Assuming all adapters support the same models**: Each adapter's model list is specific to its provider. Check the metadata before configuring.

## Best Practices

- **Use opencode_local when you need provider flexibility**: If your team uses multiple AI providers, `opencode_local` eliminates the need to configure multiple adapter types.
- **Pin model versions in production**: Use specific model version strings instead of aliases to ensure consistent behavior across runs.
- **Test adapter connectivity before deploying agents**: Use the adapter's `test.ts` validation to verify API keys and CLI installations before creating agents.

## Summary

- The `codex_local` adapter invokes OpenAI Codex CLI and supports dynamic model discovery
- The `gemini_local` adapter targets Google Gemini CLI with multimodal model options
- The `opencode_local` adapter supports multiple providers via `provider/model` format
- Each adapter uses the same architecture (metadata, execute, three modules) but with provider-specific configuration
- Choose adapters based on model capabilities, API access, and flexibility requirements

## Code Examples

**The opencode_local adapter uses provider/model format to support multiple AI providers through a single CLI interface**

```typescript
// OpenCode adapter supports multiple providers
export const metadata: AdapterMetadata = {
  type: "opencode_local",
  label: "OpenCode (Local)",
  models: [
    "anthropic/claude-sonnet-4-20250514",
    "openai/gpt-4o",
    "google/gemini-2.5-pro"
  ],
  agentConfigurationDoc: "https://paperclip.ing/docs/adapters/opencode-local"
};
```


## Resources

- [Paperclip Codex Adapter Documentation](https://paperclip.ing/docs/adapters/codex-local) — Official documentation for configuring the Codex local adapter
- [Paperclip Adapters Overview](https://paperclip.ing/docs/adapters) — Overview of all built-in adapters and their capabilities

---

> 📘 *This lesson is part of the [Paperclip Adapters & Agent Runtimes](https://stanza.dev/courses/paperclip-adapters) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*