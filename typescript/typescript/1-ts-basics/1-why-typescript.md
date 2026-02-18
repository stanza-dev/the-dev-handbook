---
source_course: "typescript"
source_lesson: "typescript-why-typescript"
---

# Why TypeScript?

TypeScript is a superset of JavaScript that adds static typing. This means you can catch errors at compile-time rather than runtime.

```typescript
function greet(name: string) {
  console.log("Hello, " + name);
}

// greet(42); // Error: Argument of type 'number' is not assignable to parameter of type 'string'.
```

## Benefits
- **Safety**: Catch bugs early (like typos or passing the wrong type).
- **Tooling**: Better autocompletion, navigation, and refactoring in your IDE.
- **Readability**: Types serve as documentation that never goes out of date.

## The Compiler
Browsers understand JavaScript, not TypeScript. You must compile (or "transpile") TS into JS using the TypeScript compiler (`tsc`).

### Resources
- [TypeScript Handbook: The Basics](https://www.typescriptlang.org/docs/handbook/2/basic-types.html)

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*