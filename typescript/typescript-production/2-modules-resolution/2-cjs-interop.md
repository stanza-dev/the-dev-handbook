---
source_course: "typescript-production"
source_lesson: "typescript-production-esm-cjs-interop"
---

# ESM & CJS Interop

## Introduction
The JavaScript ecosystem is split between CommonJS (CJS) and ECMAScript Modules (ESM). TypeScript must navigate both systems, especially when CJS libraries are consumed by ESM code and vice versa. Understanding the interop rules prevents mysterious "default import" errors and module format mismatches.

## Key Concepts
- **ESM (ECMAScript Modules)**: The standard module system using `import`/`export`. Files use `.mjs` or `.mts` extension, or `"type": "module"` in package.json.
- **CJS (CommonJS)**: The legacy Node.js module system using `require()`/`module.exports`. Files use `.cjs` or `.cts` extension.
- **esModuleInterop**: A tsconfig flag that generates helper code to smooth CJS-to-ESM default import differences.

## Real World Context
You are writing an ESM Node.js app but depend on `express`, which is CommonJS. Without `esModuleInterop`, `import express from "express"` fails because CJS modules do not have a default export. With it, TypeScript generates a helper that wraps the CJS export.

## Deep Dive
### File Extensions

TypeScript uses file extensions to determine module format:

| Extension | Module Format | TypeScript Equivalent |
|-----------|--------------|----------------------|
| `.mjs` | ESM | `.mts` |
| `.cjs` | CJS | `.cts` |
| `.js` | Depends on package.json `"type"` | `.ts` |

### esModuleInterop

```json
{ "esModuleInterop": true }
```

Without this flag:
```typescript
// CJS module: module.exports = function() {}
import express from "express"; // Error: no default export
import * as express from "express"; // Works but ugly
```

With this flag:
```typescript
import express from "express"; // Works — helper wraps CJS export
```

### Dual Publishing

Libraries that support both CJS and ESM use conditional exports in package.json:

```json
{
    "exports": {
        ".": {
            "import": "./dist/esm/index.mjs",
            "require": "./dist/cjs/index.cjs",
            "types": "./dist/types/index.d.ts"
        }
    }
}
```

### require() of ESM (Node.js 22+ / TypeScript 5.8+)

Node.js 22 introduced synchronous `require()` of ESM modules. TypeScript 5.8+ recognizes this:

```typescript
// In a .cts file, you can now require() an ESM module
const { z } = require("zod"); // Works if zod is ESM-only
```

## Common Pitfalls
1. **Missing esModuleInterop** — Without it, default imports from CJS modules fail. Always enable this flag.
2. **Incorrect "type" in package.json** — If `"type": "module"` is set, all `.js` files are treated as ESM. CJS files must use `.cjs`.

## Best Practices
1. **Always enable esModuleInterop** — It is enabled by default in new tsconfigs and fixes the most common interop issue.
2. **Use .mts/.cts for explicit format** — When you need to mix ESM and CJS in the same project, use explicit extensions.

## Summary
- ESM uses `import`/`export`; CJS uses `require()`/`module.exports`.
- `esModuleInterop` fixes default import issues when consuming CJS from ESM.
- File extensions (`.mts`, `.cts`) explicitly control module format.
- Dual publishing uses conditional exports in package.json.

## Code Examples

**Dual-publish package.json with conditional exports — ESM consumers get .mjs, CJS consumers get .cjs, both get types**

```json
{
  "name": "my-library",
  "type": "module",
  "exports": {
    ".": {
      "import": {
        "types": "./dist/index.d.mts",
        "default": "./dist/index.mjs"
      },
      "require": {
        "types": "./dist/index.d.cts",
        "default": "./dist/index.cjs"
      }
    }
  }
}
```


## Resources

- [Node.js Modules Documentation](https://nodejs.org/api/esm.html) — Official Node.js documentation on ECMAScript Modules and interop with CommonJS

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*