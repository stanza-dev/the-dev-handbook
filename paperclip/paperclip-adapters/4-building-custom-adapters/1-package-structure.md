---
source_course: "paperclip-adapters"
source_lesson: "paperclip-adapters-package-structure"
---

# Adapter Package Structure

## Introduction

Building a custom Paperclip adapter starts with understanding the directory layout. Every adapter — built-in or custom — follows the same structure: a root metadata file, a server module, a UI module, and a CLI module. Getting this structure right ensures that Paperclip can discover, load, and use your adapter across all contexts.

## Key Concepts

- **Root Metadata**: The `src/index.ts` file that exports the adapter's type key, label, supported models, and documentation link
- **Server Directory**: Contains `execute.ts` for core execution logic and `test.ts` for environment validation
- **UI Directory**: Contains components for transcript rendering, configuration forms, and config building
- **CLI Directory**: Contains terminal formatting logic for watch mode
- **Dependency-Free Requirement**: The root metadata and server module must not depend on external packages beyond Paperclip's adapter utilities
- **Three Registries**: Adapters register separately in server, UI, and CLI registries

## Real World Context

A machine learning team has a custom inference server built on vLLM. None of Paperclip's built-in adapters support vLLM's specific API. The team decides to build a custom adapter. By following the standard directory layout, their adapter integrates seamlessly with Paperclip's UI, CLI, and server — users configure it through the same forms, view transcripts in the same run viewer, and monitor it in the same dashboard.

## Deep Dive

Here is the directory layout for a custom adapter:

```typescript
// Custom adapter directory structure
// adapters/vllm-adapter/
//   src/
//     index.ts           ← Root metadata
//     server/
//       execute.ts       ← Core execution logic
//       test.ts          ← Environment validation
//     ui/
//       parse-stdout.ts  ← Transcript rendering
//       build-config.ts  ← Form input → config JSON
//       config-fields.tsx ← React form component
//     cli/
//       format-event.ts  ← Terminal formatting
```

The root `index.ts` file exports the adapter metadata. This is the first thing Paperclip reads when discovering your adapter:

```typescript
import type { AdapterMetadata } from "@paperclipai/adapter-core";

export const metadata: AdapterMetadata = {
  type: "vllm_custom",
  label: "vLLM (Custom)",
  models: ["llama-3-70b", "mistral-7b", "codellama-34b"],
  agentConfigurationDoc: "https://internal.docs/adapters/vllm"
};
```

The `type` field must be unique across all registered adapters. The `label` appears in the UI's adapter selector dropdown. The `models` array populates the model selection field in configuration forms. And `agentConfigurationDoc` links to your adapter's documentation.

A critical requirement is that the root metadata file must remain **dependency-free**. It should only import types from `@paperclipai/adapter-core`. This restriction exists because the metadata is loaded in multiple contexts (server, UI, CLI), and external dependencies could break in any of them:

```typescript
// CORRECT: only type imports from adapter-core
import type { AdapterMetadata } from "@paperclipai/adapter-core";

// INCORRECT: importing an external package
// import axios from "axios"; // DO NOT do this in index.ts
```

Once your adapter structure is in place, you register it in all three registries:

```typescript
// Register in server registry
import { registerServerAdapter } from "@paperclipai/adapter-registry/server";
import * as vllmServer from "./adapters/vllm-adapter/src/server";
registerServerAdapter("vllm_custom", vllmServer);

// Register in UI registry
import { registerUIAdapter } from "@paperclipai/adapter-registry/ui";
import * as vllmUI from "./adapters/vllm-adapter/src/ui";
registerUIAdapter("vllm_custom", vllmUI);

// Register in CLI registry
import { registerCLIAdapter } from "@paperclipai/adapter-registry/cli";
import * as vllmCLI from "./adapters/vllm-adapter/src/cli";
registerCLIAdapter("vllm_custom", vllmCLI);
```

All three registrations use the same type key (`vllm_custom`). This key ties the server execution logic, UI components, and CLI formatters together as a single adapter.

## Common Pitfalls

- **Adding dependencies to index.ts**: The root metadata file must be dependency-free. Any imports beyond `@paperclipai/adapter-core` types will break adapter discovery.
- **Using a non-unique type key**: If your type key conflicts with a built-in adapter, Paperclip will overwrite the built-in registration. Always check existing type keys before choosing yours.
- **Forgetting to register in all three registries**: An adapter registered only in the server registry will execute correctly but will not render transcripts or format CLI output.

## Best Practices

- **Copy a built-in adapter as a starting point**: The claude_local adapter is a well-documented example. Clone its directory structure and modify the implementation.
- **Use descriptive type keys**: Include your organization or use case in the key, like `mycompany_vllm` or `internal_llama`, to avoid future conflicts.
- **Keep the three modules independent**: The server module should never import from the UI module. Each module is loaded in isolation in its respective context.

## Summary

- Every custom adapter follows the same directory structure: root metadata, server, UI, and CLI modules
- The root `index.ts` exports metadata including type key, label, models, and documentation link
- The root metadata file must remain dependency-free — only import types from `@paperclipai/adapter-core`
- Adapters are registered in three separate registries (server, UI, CLI) using the same type key
- The three modules are loaded independently and must not import from each other

## Code Examples

**Root metadata file for a custom adapter — must remain dependency-free and only import types from @paperclipai/adapter-core**

```typescript
import type { AdapterMetadata } from "@paperclipai/adapter-core";

export const metadata: AdapterMetadata = {
  type: "vllm_custom",
  label: "vLLM (Custom)",
  models: ["llama-3-70b", "mistral-7b", "codellama-34b"],
  agentConfigurationDoc: "https://internal.docs/adapters/vllm"
};
```


## Resources

- [Paperclip Custom Adapters Guide](https://paperclip.ing/docs/adapters/custom) — Official guide for building custom Paperclip adapters from scratch
- [Paperclip GitHub — Built-in Adapters](https://github.com/paperclipai/paperclip) — Source code for built-in adapters to use as templates for custom implementations

---

> 📘 *This lesson is part of the [Paperclip Adapters & Agent Runtimes](https://stanza.dev/courses/paperclip-adapters) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*