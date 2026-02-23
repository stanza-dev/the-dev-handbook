---
source_course: "typescript-production"
source_lesson: "typescript-production-import-defer"
---

# Deferred Imports (import defer)

## Introduction
TypeScript 5.9 adds support for the `import defer` proposal (TC39 Stage 2), which allows lazy evaluation of imported modules. The module is fetched and linked at import time but its code does not execute until you first access one of its exports. This can significantly improve startup performance for applications with heavy dependencies.

## Key Concepts
- **import defer**: A new import syntax that delays module execution until first property access.
- **Lazy Evaluation**: The module file is loaded into memory but its top-level code does not run until an export is accessed.
- **Tree Shaking Complement**: While tree shaking removes unused code, deferred imports delay execution of used code.

## Real World Context
A CLI tool that imports a large charting library but only uses it when the user runs a specific command. With `import defer`, the charting library does not execute at startup — only when the charting command is invoked. This can shave seconds off startup time.

## Deep Dive
### Syntax

```typescript
import defer * as mathLib from "heavy-math-library";

// At this point, heavy-math-library has NOT executed
console.log("App started"); // Fast startup

// Module executes on first property access
const result = mathLib.calculate(42); // NOW it executes
```

The `defer` keyword goes between `import` and the binding. It requires a namespace import (`* as name`) — you cannot use named imports with defer.

### How It Differs from Dynamic import()

```typescript
// Dynamic import — async, returns a Promise
const mathLib = await import("heavy-math-library");

// Deferred import — synchronous, no await needed
import defer * as mathLib from "heavy-math-library";
mathLib.calculate(42); // Synchronous access, no await
```

The key difference: `import defer` is synchronous. You do not need `await` or `.then()`. The module is already loaded and linked — only execution is deferred.

### When to Use

Deferred imports are most useful for:
- **Large utility libraries** used conditionally (logging, profiling, charting)
- **CLI tools** where most commands use only a subset of dependencies
- **Development-only code** (test utilities, debug tools) that should not slow production startup

### Limitations

- Requires `--module nodenext` or `--module preserve` in tsconfig
- Only works with namespace imports (`* as name`), not named imports
- The module must be side-effect free for deferred execution to be safe

## Common Pitfalls
1. **Side effects in deferred modules** — If the module runs setup code at the top level (polyfills, global state), deferring it may cause bugs because that code runs later than expected.
2. **Using named imports** — `import defer { calc } from "..."` is not valid syntax. You must use `import defer * as lib from "..."`.

## Best Practices
1. **Use for heavy, conditionally-used dependencies** — The biggest wins come from deferring large libraries that are not always needed.
2. **Ensure modules are side-effect free** — Only defer modules whose top-level code has no observable side effects.

## Summary
- `import defer` delays module execution until first property access.
- It is synchronous unlike dynamic `import()` — no await needed.
- Best for heavy, conditionally-used dependencies to improve startup time.
- Requires namespace imports and side-effect-free modules.

## Code Examples

**import defer delays the analytics module execution until generateReport is actually called — startup stays fast**

```typescript
// Deferred import — module loads but does NOT execute yet
import defer * as analytics from "./heavy-analytics";

// Fast startup — analytics code hasn't run
console.log("App ready");

function handleUserAction(action: string) {
    if (action === "view-report") {
        // NOW the analytics module executes (first access)
        analytics.generateReport();
    }
    // If the user never views a report,
    // the analytics module never executes
}
```


## Resources

- [TC39 Deferred Module Evaluation Proposal](https://github.com/tc39/proposal-defer-import-eval) — The TC39 Stage 2 proposal for deferred module evaluation that TypeScript 5.9 supports

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*