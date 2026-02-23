---
source_course: "typescript"
source_lesson: "typescript-why-typescript"
---

# Why TypeScript?

## Introduction
TypeScript is a superset of JavaScript that adds static typing, enabling you to catch errors at compile time rather than discovering them at runtime. If you have ever shipped a bug caused by a typo or a wrong argument type, TypeScript exists to prevent exactly that.

## Key Concepts
- **Static Typing**: Types are checked before the code runs, during compilation.
- **Type Inference**: TypeScript can automatically determine types from context, so you do not need to annotate everything.
- **Transpilation**: TypeScript code is compiled into plain JavaScript that browsers and Node.js can execute.

## Real World Context
Every major frontend framework (React, Angular, Vue) and most backend Node.js projects now ship with TypeScript support. Teams adopt TypeScript because it reduces bugs in production, makes refactoring safer, and serves as living documentation that IDEs can use for autocompletion.

## Deep Dive
Consider a simple JavaScript function that greets a user:

```typescript
function greet(name: string): string {
  return `Hello, ${name}!`;
}
```

The `: string` annotations tell TypeScript two things: the `name` parameter must be a string, and the function will return a string. If someone tries to call `greet(42)`, the compiler flags the error immediately.

TypeScript also catches property access mistakes:

```typescript
const user = { name: "Alice", role: "admin" };
// user.rle; // Error: Property 'rle' does not exist — did you mean 'role'?
```

The compiler sees the typo and suggests a fix before the code ever runs.

Because TypeScript is a superset of JavaScript, every valid JavaScript file is also valid TypeScript. You can adopt it incrementally by renaming `.js` files to `.ts` and adding types gradually.

## Common Pitfalls
1. **Over-annotating everything** — TypeScript has powerful inference. Writing `const count: number = 5` is redundant because TypeScript already knows `5` is a number. Let inference do its job and annotate only where necessary.
2. **Using `any` to silence errors** — Reaching for `any` defeats the purpose of TypeScript. When you encounter a type error, fix the type instead of escaping the type system.

## Best Practices
1. **Enable strict mode** — The `strict` flag in `tsconfig.json` turns on all strict type-checking options. Start strict from day one; it is much harder to retrofit later.
2. **Let inference work for you** — Annotate function parameters and return types, but let TypeScript infer local variables. This keeps code concise without losing safety.

## Summary
- TypeScript adds static types to JavaScript, catching errors at compile time.
- It is a superset of JavaScript, so adoption can be incremental.
- Strong tooling support (autocompletion, refactoring, error detection) makes developers more productive.

## Code Examples

**TypeScript prevents passing a string where a number is expected, catching the bug at compile time instead of producing NaN at runtime**

```typescript
// TypeScript catches type errors before runtime
function calculateTotal(price: number, quantity: number): number {
  return price * quantity;
}

const total = calculateTotal(29.99, 3); // 89.97
// calculateTotal("29.99", 3); // Error: Argument of type 'string' is not assignable to parameter of type 'number'
```


## Resources

- [TypeScript Handbook: The Basics](https://www.typescriptlang.org/docs/handbook/2/basic-types.html) — Official introduction to TypeScript's type system
- [TypeScript for JavaScript Programmers](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html) — A quick overview of TypeScript for developers coming from JavaScript

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*