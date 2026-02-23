---
source_course: "typescript"
source_lesson: "typescript-enums"
---

# Enums

## Introduction
Enums let you define a set of named constants, giving meaningful labels to values that would otherwise be magic numbers or strings scattered through your code. TypeScript provides both numeric and string-based enums, plus a modern alternative using `as const`.

## Key Concepts
- **Numeric Enum**: An enum where members auto-increment from 0 (or a custom start value).
- **String Enum**: An enum where each member has an explicit string value.
- **`as const` Assertion**: A modern alternative that creates a readonly object with literal types, often preferred over enums.
- **Reverse Mapping**: Numeric enums allow looking up the name from a value, but string enums do not.

## Real World Context
Enums are common in APIs and configuration. HTTP status codes, user roles, and order states are natural fits. However, many modern TypeScript codebases have shifted to `as const` objects because they produce cleaner JavaScript output and work better with tree-shaking.

## Deep Dive

### Numeric Enums

By default, enums are zero-indexed:

```typescript
enum Priority {
  Low,     // 0
  Medium,  // 1
  High,    // 2
  Critical // 3
}

const taskPriority: Priority = Priority.High; // 2
```

Numeric enums support reverse mapping, meaning you can look up the name from the value:

```typescript
console.log(Priority[2]); // "High"
```

### String Enums

String enums require explicit values but produce more readable output:

```typescript
enum OrderStatus {
  Pending = "PENDING",
  Processing = "PROCESSING",
  Shipped = "SHIPPED",
  Delivered = "DELIVERED",
}

function updateOrder(status: OrderStatus): void {
  console.log(`Order is now: ${status}`);
}

updateOrder(OrderStatus.Shipped); // "Order is now: SHIPPED"
```

String enums are preferred over numeric enums because the values are self-documenting and less error-prone.

### The `as const` Alternative

Many modern codebases use `as const` objects instead of enums:

```typescript
const ROLES = {
  Admin: "admin",
  Editor: "editor",
  Viewer: "viewer",
} as const;

type Role = (typeof ROLES)[keyof typeof ROLES]; // "admin" | "editor" | "viewer"

function assignRole(role: Role): void {
  console.log(`Assigning role: ${role}`);
}

assignRole(ROLES.Admin); // OK
assignRole("editor");    // Also OK — it is a literal type
```

The `as const` approach has several advantages: it produces no extra runtime code (enums compile to objects with reverse mappings), it works seamlessly with type inference, and the values are just plain strings that can be used interchangeably.

### When to Use Which

Use string enums when you need a closed set of values that are used with the `enum.Member` syntax. Use `as const` when you want plain string values that work in both TypeScript types and runtime JavaScript with no overhead.

## Common Pitfalls
1. **Using numeric enums for API values** — If your API returns `"PENDING"`, a numeric enum with value `0` will not match. Use string enums or `as const` for values that cross API boundaries.
2. **Assuming enums are tree-shakeable** — Enums compile to objects that bundlers cannot easily remove. For library code, prefer `as const` for smaller bundle sizes.

## Best Practices
1. **Prefer string enums over numeric enums** — String values are self-documenting and do not break when members are reordered.
2. **Consider `as const` for new code** — It is lighter, more flexible, and produces cleaner JavaScript output.

## Summary
- Numeric enums auto-increment from 0; string enums require explicit values.
- `as const` objects are a modern, lightweight alternative to enums.
- String enums and `as const` produce clearer code than numeric enums for most use cases.

## Code Examples

**An 'as const' object that acts like an enum but produces no extra runtime code and works with plain string values**

```typescript
// Modern alternative to enums using 'as const'
const THEME = {
  Light: "light",
  Dark: "dark",
  System: "system",
} as const;

type Theme = (typeof THEME)[keyof typeof THEME]; // "light" | "dark" | "system"

function applyTheme(theme: Theme): void {
  document.documentElement.setAttribute("data-theme", theme);
}

applyTheme(THEME.Dark); // OK
applyTheme("light");    // Also OK
```


## Resources

- [TypeScript Handbook: Enums](https://www.typescriptlang.org/docs/handbook/enums.html) — Official reference for numeric, string, and const enums

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*