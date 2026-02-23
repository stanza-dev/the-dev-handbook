---
source_course: "typescript"
source_lesson: "typescript-type-annotations"
---

# Type Annotations

## Introduction
Type annotations are how you explicitly tell TypeScript what type a variable, parameter, or return value should be. They are the most fundamental building block of the type system and the first thing you will write when moving from JavaScript to TypeScript.

## Key Concepts
- **Type Annotation**: A colon followed by a type name (e.g., `: string`) placed after a variable, parameter, or function signature.
- **Return Type Annotation**: Placed after the parameter list to declare what a function returns.
- **Type Inference**: When TypeScript can figure out the type on its own, the annotation is optional.

## Real World Context
When writing API handlers, you annotate request and response types so that your IDE can autocomplete property names and catch mistakes. For example, annotating a user object ensures you never accidentally access `user.emial` instead of `user.email`.

## Deep Dive

### Variable Annotations

You place the type after the variable name with a colon:

```typescript
let username: string = "Alice";
let age: number = 30;
let isActive: boolean = true;
```

These annotations are optional here because TypeScript can infer the types from the assigned values. However, they become essential when declaring variables without an initial value:

```typescript
let score: number;
// score is 'number', not 'any'
score = 100;
```

Without the annotation, `score` would be inferred as `any`, which removes all type safety.

### Parameter Annotations

Function parameters should always be annotated because TypeScript cannot infer their types from the function definition alone:

```typescript
function createEmail(to: string, subject: string, body: string): string {
  return `To: ${to}\nSubject: ${subject}\n\n${body}`;
}
```

The `: string` after each parameter name tells TypeScript what callers must pass. The `: string` after the closing parenthesis is the return type.

### Return Type Annotations

While TypeScript can infer return types, explicitly annotating them is recommended for public functions because it serves as documentation and prevents accidental return type changes:

```typescript
function divide(a: number, b: number): number {
  return a / b;
}
```

If you accidentally returned a string from this function, TypeScript would flag the mismatch immediately.

## Common Pitfalls
1. **Annotating when inference suffices** — Writing `const name: string = "Alice"` is redundant. TypeScript already knows the type from the literal. Reserve annotations for parameters, uninitialized variables, and complex return types.
2. **Using uppercase type names** — `String`, `Number`, and `Boolean` are wrapper objects, not primitives. Always use lowercase: `string`, `number`, `boolean`.

## Best Practices
1. **Always annotate function parameters** — Parameters have no inference source, so annotations are required for type safety.
2. **Annotate return types on exported functions** — This prevents accidental changes to the function's contract and improves readability for consumers.

## Summary
- Type annotations use the `: Type` syntax after names.
- Always annotate function parameters; return types are recommended for public APIs.
- Use lowercase primitive types (`string`, not `String`).

## Code Examples

**Annotating parameters and return types ensures callers pass the correct types and consumers know what to expect**

```typescript
// Parameter and return type annotations
function formatCurrency(amount: number, currency: string): string {
  return `${currency} ${amount.toFixed(2)}`;
}

const price = formatCurrency(19.99, "USD"); // "USD 19.99"
// formatCurrency("19.99", "USD"); // Error: 'string' is not assignable to 'number'
```


## Resources

- [TypeScript Handbook: Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html) — Covers type annotations for variables, functions, and objects

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*