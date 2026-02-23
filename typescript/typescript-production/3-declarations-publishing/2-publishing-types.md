---
source_course: "typescript-production"
source_lesson: "typescript-production-publishing-types"
---

# Publishing Typed Libraries

## Introduction
Publishing a TypeScript library that works seamlessly for consumers requires correct `package.json` configuration, proper `exports` fields, and declaration file generation. Get this wrong and your users face "cannot find module" errors or lose type information entirely.

## Key Concepts
- **package.json exports**: The modern way to define what files consumers can import from your package.
- **types condition**: A special condition in exports that tells TypeScript where to find declaration files.
- **typesVersions**: A fallback mechanism for consumers on older TypeScript versions.

## Real World Context
When you publish `my-utils` to npm and a user writes `import { sort } from "my-utils"`, TypeScript needs to find the `.d.ts` file for that import. The `exports` field in package.json tells TypeScript (and Node.js) exactly where to look.

## Deep Dive
### Modern Exports Configuration

```json
{
    "name": "my-utils",
    "type": "module",
    "exports": {
        ".": {
            "types": "./dist/index.d.ts",
            "import": "./dist/index.js",
            "require": "./dist/index.cjs"
        },
        "./sorting": {
            "types": "./dist/sorting.d.ts",
            "import": "./dist/sorting.js"
        }
    }
}
```

The `types` condition MUST come first — TypeScript processes conditions in order and uses the first match.

### Subpath Exports

Allow consumers to import submodules:

```typescript
import { sort } from "my-utils";          // Main entry
import { quickSort } from "my-utils/sorting"; // Subpath
```

### Legacy Fields (Fallback)

For consumers on older TypeScript or Node.js:

```json
{
    "main": "./dist/index.cjs",
    "module": "./dist/index.js",
    "types": "./dist/index.d.ts"
}
```

### Build Configuration

A typical build setup with `tsup` (which uses esbuild):

```bash
tsup src/index.ts --format cjs,esm --dts
```

This generates:
- `dist/index.js` (ESM)
- `dist/index.cjs` (CJS)
- `dist/index.d.ts` (declarations)

## Common Pitfalls
1. **types condition not first** — If `import` comes before `types` in the exports object, TypeScript may skip the types entry. Always put `types` first.
2. **Missing declaration files in published package** — If your `.d.ts` files are not included in the npm publish, consumers get no types. Check your `files` field in package.json.

## Best Practices
1. **Use the exports field** — It is the modern standard and supports both ESM and CJS consumers.
2. **Include types in your files array** — Ensure `"files": ["dist"]` includes both JS and `.d.ts` output.

## Summary
- Use `package.json` `exports` with a `types` condition (listed first) for typed libraries.
- Support subpath exports for granular imports.
- Use tools like tsup to generate CJS, ESM, and declaration files in one step.
- Always verify `.d.ts` files are included in the published package.

## Code Examples

**Complete package.json for a dual CJS/ESM library — types condition comes first in exports for TypeScript resolution**

```json
{
  "name": "my-utils",
  "version": "1.0.0",
  "type": "module",
  "files": ["dist"],
  "exports": {
    ".":{
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    }
  },
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts"
  }
}
```


## Resources

- [TypeScript: Publishing Declaration Files](https://www.typescriptlang.org/docs/handbook/declaration-files/publishing.html) — Official guide on publishing TypeScript declaration files with npm packages

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*