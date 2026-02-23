---
source_course: "typescript"
source_lesson: "typescript-satisfies-operator"
---

# The satisfies Operator

## Introduction
Introduced in TypeScript 4.9, the `satisfies` operator lets you validate that a value matches a type without widening it. Unlike type annotations (which can lose specificity) and type assertions (which override the compiler), `satisfies` checks the type while preserving the most specific type TypeScript can infer.

## Key Concepts
- **`satisfies` Operator**: Validates a value against a type at the declaration site without changing the inferred type.
- **Type Widening**: When a type annotation causes TypeScript to forget specific literal types (e.g., `"red"` becomes `string`).
- **Validation Without Widening**: `satisfies` ensures the value conforms to a type while keeping the narrowest inferred type.

## Real World Context
Configuration objects are the canonical use case. You want to ensure a config object matches a schema (has the right keys and value types) but also want to keep the literal types for autocompletion. With `satisfies`, your IDE shows the exact values while the compiler still validates the shape.

## Deep Dive

### The Problem: Annotation Widens, Assertion Lies

Consider a color palette:

```typescript
type ColorConfig = Record<string, string | number[]>;

// With annotation — loses specificity
const palette: ColorConfig = {
  primary: "#3178c6",
  secondary: "#6b7280",
  accent: [49, 120, 198],
};

palette.primary.toUpperCase(); // Error: Property 'toUpperCase' does not exist on 'string | number[]'
```

The annotation `ColorConfig` widens every value to `string | number[]`, so TypeScript does not know that `primary` is specifically a `string`.

### The Solution: `satisfies`

```typescript
const palette = {
  primary: "#3178c6",
  secondary: "#6b7280",
  accent: [49, 120, 198],
} satisfies ColorConfig;

palette.primary.toUpperCase();  // OK — TypeScript knows primary is "#3178c6"
palette.accent.map(v => v * 2); // OK — TypeScript knows accent is number[]
```

The `satisfies` operator validates that the object matches `ColorConfig` (correct keys, correct value types) but does not widen the inferred types. Each property retains its specific type.

### Catching Errors While Keeping Specificity

`satisfies` catches structural errors just like an annotation would:

```typescript
const palette = {
  primary: "#3178c6",
  secondary: "#6b7280",
  accent: true, // Error: Type 'boolean' is not assignable to 'string | number[]'
} satisfies ColorConfig;
```

The boolean `true` does not match `string | number[]`, so the compiler flags it immediately.

### Combining with `as const`

For maximum specificity, combine `satisfies` with `as const`:

```typescript
type Route = { path: string; method: "GET" | "POST" };

const routes = {
  home: { path: "/", method: "GET" },
  login: { path: "/login", method: "POST" },
} as const satisfies Record<string, Route>;

// routes.home.path is "/" (literal), not string
// routes.login.method is "POST" (literal), not "GET" | "POST"
// Still validated against Record<string, Route>
```

The `as const` makes all values literal and readonly, while `satisfies` ensures the object matches the expected shape.

## Common Pitfalls
1. **Using `satisfies` when an annotation is fine** — If you do not need to preserve literal types, a simple annotation works. `satisfies` adds value when you need both validation and specificity.
2. **Confusing `satisfies` with `as`** — `satisfies` validates; `as` overrides. They serve different purposes. Use `satisfies` when you want the compiler to check the type. Use `as` when you want to override the compiler.

## Best Practices
1. **Use `satisfies` for configuration objects and constants** — Any object that should match a schema but needs to retain literal types for autocompletion.
2. **Combine `as const satisfies Type` for maximum safety** — This pattern gives you readonly literal types with full schema validation.

## Summary
- `satisfies` validates a value against a type without widening the inferred type.
- It catches structural errors while preserving literal types for autocompletion.
- Combine with `as const` for readonly, literal, and validated types.
- Use it for configuration objects, theme definitions, and route tables.

## Code Examples

**The satisfies operator ensures the theme matches the Theme type while preserving the exact literal values for IDE autocompletion**

```typescript
// satisfies validates the shape without losing literal types
type Theme = {
  colors: Record<string, string>;
  spacing: Record<string, number>;
};

const theme = {
  colors: {
    primary: "#3178c6",
    background: "#ffffff",
  },
  spacing: {
    sm: 4,
    md: 8,
    lg: 16,
  },
} satisfies Theme;

// theme.colors.primary is "#3178c6" (literal), not just string
// theme.spacing.md is 8 (literal), not just number
```


## Resources

- [TypeScript 4.9: The satisfies Operator](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html#the-satisfies-operator) — Official release notes explaining the satisfies operator

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*