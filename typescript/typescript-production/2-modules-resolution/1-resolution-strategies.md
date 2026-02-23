---
source_course: "typescript-production"
source_lesson: "typescript-production-module-resolution-strategies"
---

# Module Resolution Strategies

## Introduction
Module resolution is how TypeScript decides what file an `import` statement refers to. The strategy you choose determines whether TypeScript requires file extensions, respects `package.json` exports, and how it searches `node_modules`. TypeScript 5.9 introduced new strategies that align with modern Node.js ESM behavior.

## Key Concepts
- **Module Resolution Strategy**: The algorithm TypeScript uses to map an import specifier to a file on disk.
- **node10 (legacy Node)**: The classic Node.js CommonJS resolution — checks `index.js`, allows extensionless imports.
- **node16 / nodenext**: ESM-aware resolution that requires file extensions and respects `package.json` `exports`.
- **bundler**: For Vite, Webpack, esbuild — understands ESM imports but delegates final resolution to the bundler.

## Real World Context
The Node.js ecosystem is migrating from CommonJS to ESM. TypeScript's resolution strategies must match your runtime and toolchain. Using the wrong strategy causes "module not found" errors that are confusing because the file clearly exists — TypeScript just was not looking for it correctly.

## Deep Dive
### Strategy Comparison

| Strategy | Extensions Required | Respects `exports` | Best For |
|----------|-------------------|-------------------|----------|
| `node10` (Node) | No | No | Legacy CJS projects |
| `node16` / `nodenext` | Yes (`.js`) | Yes | Modern Node.js (ESM) |
| `bundler` | No | Yes | Vite, Webpack, esbuild |

### node10 (Legacy)

```json
{ "moduleResolution": "Node" }
```

This is the classic strategy. It checks `./file.ts`, `./file/index.ts`, and `node_modules` without requiring extensions. It does NOT respect `package.json` `exports` fields.

### node16 / nodenext

```json
{ "module": "NodeNext", "moduleResolution": "NodeNext" }
```

This strategy mirrors modern Node.js behavior:

```typescript
// Must use .js extension (even though the source is .ts)
import { helper } from "./utils.js";

// Package imports respect the "exports" field
import { z } from "zod"; // Resolves via zod's package.json exports
```

The `.js` extension requirement catches developers off-guard: you write `.ts` files but import with `.js` extensions because TypeScript resolves them to the corresponding `.ts` file during compilation.

### bundler

```json
{ "module": "ESNext", "moduleResolution": "Bundler" }
```

For bundler-based projects (Vite, Webpack):
- No file extensions required in imports
- Respects `package.json` `exports`
- Does not enforce Node.js-specific rules

### Path Mapping

Map custom import paths to directories:

```json
{
    "compilerOptions": {
        "baseUrl": ".",
        "paths": {
            "@utils/*": ["src/utils/*"],
            "@components/*": ["src/components/*"]
        }
    }
}
```

```typescript
import { formatDate } from "@utils/date"; // Resolves to src/utils/date.ts
```

## Common Pitfalls
1. **Using node10 with ESM packages** — Modern packages with `exports` fields will not resolve correctly with the legacy strategy.
2. **Forgetting .js extensions with nodenext** — TypeScript requires `.js` extensions in import paths even for `.ts` source files. This is by design to match Node.js runtime behavior.

## Best Practices
1. **Match strategy to your runtime** — Use `NodeNext` for Node.js apps, `Bundler` for frontend projects with Vite/Webpack.
2. **Always pair module and moduleResolution** — `"module": "NodeNext"` requires `"moduleResolution": "NodeNext"`. Mismatches cause confusing errors.

## Summary
- `node10` is the legacy strategy for CommonJS — no extensions, no exports support.
- `node16`/`nodenext` requires `.js` extensions and respects `package.json` `exports`.
- `bundler` is ideal for Vite/Webpack — no extensions required but exports are respected.
- Always pair `module` and `moduleResolution` settings.

## Code Examples

**NodeNext requires .js extensions in imports while Bundler does not — choose based on your runtime environment**

```typescript
// With moduleResolution: "NodeNext"
// Source file: src/utils/date.ts
// Import MUST use .js extension (TypeScript resolves .js → .ts)
import { formatDate } from "./utils/date.js";

// Package imports resolve via package.json "exports"
import { z } from "zod";

// With moduleResolution: "Bundler" — no extension needed
import { formatDate } from "./utils/date";
```


## Resources

- [TypeScript Module Resolution](https://www.typescriptlang.org/docs/handbook/modules/theory.html#module-resolution) — Official docs on how TypeScript resolves module imports

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*