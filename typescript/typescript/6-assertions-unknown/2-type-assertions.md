---
source_course: "typescript"
source_lesson: "typescript-type-assertions"
---

# Type Assertions (Casting)

## Introduction
Type assertions tell the TypeScript compiler to treat a value as a specific type. Unlike narrowing (which uses runtime checks), assertions are a compile-time-only instruction that says "trust me, I know the type." They are powerful but can hide bugs if used carelessly.

## Key Concepts
- **Type Assertion (`as`)**: Tells the compiler to treat a value as a specific type without any runtime check.
- **Angle Bracket Syntax (`<Type>`)**: An older assertion syntax that conflicts with JSX and should be avoided in `.tsx` files.
- **Double Assertion**: Using `as unknown as T` to force an assertion when the types are incompatible.
- **Non-null Assertion (`!`)**: A postfix operator that tells TypeScript a value is not `null` or `undefined`.

## Real World Context
When working with the DOM, `document.getElementById()` returns `HTMLElement | null`. If you know the element exists and is a specific type, an assertion like `as HTMLCanvasElement` gives you access to canvas-specific methods. In testing, assertions help create mock objects that satisfy type requirements without implementing every property.

## Deep Dive

### The `as` Keyword

The most common assertion syntax:

```typescript
const canvas = document.getElementById("gameCanvas") as HTMLCanvasElement;
const ctx = canvas.getContext("2d"); // TypeScript knows this is a canvas element
```

Without the assertion, `canvas` would be `HTMLElement | null`, and `getContext` would not be available.

### When Assertions Are Allowed

TypeScript only allows assertions between compatible types:

```typescript
const value = "hello" as string; // OK — string to string
const count = "hello" as unknown as number; // OK — via unknown
// const invalid = "hello" as number; // Error: types are too different
```

If two types have no overlap, you must go through `unknown` first. This double assertion is a code smell — if you need it, reconsider your approach.

### Non-null Assertion

The postfix `!` operator asserts that a value is not `null` or `undefined`:

```typescript
function getLength(value: string | null): number {
  return value!.length; // Asserts value is not null
}
```

This is dangerous because there is no runtime check. If `value` is actually `null`, the code crashes. Prefer narrowing with an explicit check:

```typescript
function getLength(value: string | null): number {
  if (value === null) throw new Error("Value is null");
  return value.length; // Safely narrowed
}
```

### Const Assertions

The `as const` assertion makes values deeply readonly with literal types:

```typescript
const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3,
} as const;

// config.apiUrl is "https://api.example.com" (literal), not string
// config.timeout is 5000 (literal), not number
// config is fully readonly — no property can be reassigned
```

This is different from `as Type` assertions — `as const` does not change the type, it narrows it to the most specific version.

## Common Pitfalls
1. **Using assertions to silence errors** — An assertion hides the problem; it does not fix it. If TypeScript reports an error, understand why before reaching for `as`.
2. **Overusing non-null assertions** — The `!` operator is a promise to the compiler that you accept the risk. If the value can actually be null, you will get a runtime crash.

## Best Practices
1. **Prefer narrowing over assertions** — A `typeof` check or null guard is always safer because it verifies at runtime. Use assertions only when you have external knowledge the compiler cannot infer.
2. **Use `as const` liberally for configuration objects** — It produces the most precise types with no runtime cost.

## Summary
- Type assertions (`as Type`) tell the compiler to treat a value as a specific type.
- They are compile-time only — no runtime checking occurs.
- Non-null assertions (`!`) are risky; prefer explicit null checks.
- `as const` creates deeply readonly values with literal types.

## Code Examples

**The 'as const' assertion creates deeply readonly structures with literal types, enabling precise type extraction**

```typescript
// as const creates the narrowest possible types
const routes = [
  { path: "/", name: "Home" },
  { path: "/about", name: "About" },
  { path: "/contact", name: "Contact" },
] as const;

// routes[0].path is "/" (literal type), not string
// routes is readonly — cannot push, pop, or modify
type RoutePath = (typeof routes)[number]["path"]; // "/" | "/about" | "/contact"
```


## Resources

- [TypeScript Handbook: Everyday Types (Type Assertions)](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions) — Official guide to type assertions with the as keyword

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*