---
source_course: "typescript-production"
source_lesson: "typescript-production-module-resolution-strategies"
---

# Module Resolution

Module resolution is the process the compiler uses to figure out what an import refers to.

## Node vs Bundler

- `Node` (or `Node10`): Mimics standard Node.js lookup (looking in `node_modules`, checking `index.js`, etc.).
- `Node16` / `NodeNext`: Supports ECMAScript modules (ESM) in Node.js, respecting `exports` in `package.json` and requiring file extensions in imports.
- `Bundler`: Best for modern bundlers like Vite/Webpack. It understands TS import rules but leaves the final resolution logic (like extension guessing) to the bundler.

## Path Mapping

You can map imports to specific paths using `paths` in `compilerOptions`. This allows cleaner imports like `@utils/date` instead of `../../../utils/date`.

```json
"paths": {
  "@utils/*": ["src/utils/*"]
}
```

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*