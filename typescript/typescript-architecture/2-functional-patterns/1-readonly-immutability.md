---
source_course: "typescript-architecture"
source_lesson: "typescript-architecture-readonly-immutability"
---

# Immutability in TypeScript

## Introduction
Mutable state is the root cause of many subtle bugs — race conditions, stale closures, and unintended side effects. TypeScript provides compile-time tools to enforce immutability: `readonly` properties, `ReadonlyArray`, `Readonly<T>`, and `as const` assertions. These cost nothing at runtime but catch mutation attempts at compile time.

## Key Concepts
- **`readonly` modifier**: Prevents reassignment of object properties after initialization.
- **`ReadonlyArray<T>`**: An array type that disallows `push`, `pop`, `splice`, and index assignment.
- **`as const` assertion**: Converts an entire literal expression to its narrowest, fully-readonly form.
- **`Readonly<T>`**: A utility type that makes all properties of `T` readonly (shallow).

## Real World Context
In Redux-style state management, every state update must produce a new object rather than mutating the existing one. Marking state interfaces as `Readonly` catches accidental mutations at compile time. Configuration objects use `as const` to get literal types and prevent runtime changes.

## Deep Dive
### Readonly Properties

Mark individual properties as `readonly`:

```typescript
interface User {
    readonly id: string;
    name: string; // mutable
}

const user: User = { id: "u-1", name: "Alice" };
// user.id = "u-2"; // Error: Cannot assign to 'id' because it is a read-only property
user.name = "Bob"; // OK — name is not readonly
```

### Readonly Arrays and Tuples

Use `readonly` before array types to prevent mutation methods:

```typescript
function sum(numbers: readonly number[]): number {
    // numbers.push(10); // Error: Property 'push' does not exist on type 'readonly number[]'
    return numbers.reduce((a, b) => a + b, 0);
}
```

This is equivalent to `ReadonlyArray<number>`. For tuples:

```typescript
const pair: readonly [string, number] = ["age", 30];
// pair[0] = "name"; // Error
```

### Const Assertions

The `as const` assertion narrows literals and adds `readonly` recursively:

```typescript
const config = {
    endpoint: "https://api.example.com",
    retries: 3,
    methods: ["GET", "POST"]
} as const;

// Type is:
// {
//   readonly endpoint: "https://api.example.com";
//   readonly retries: 3;
//   readonly methods: readonly ["GET", "POST"];
// }
```

Without `as const`, `endpoint` would be `string` and `methods` would be `string[]`. With it, you get literal types and full immutability at the type level.

Important: `as const` is purely a compile-time annotation. It does NOT call `Object.freeze()` and does not prevent runtime mutation. If you need runtime immutability, use `Object.freeze()` in addition.

### Deep Readonly

`Readonly<T>` is shallow. For deep immutability, use a recursive type:

```typescript
type DeepReadonly<T> = {
    readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};
```

## Common Pitfalls
1. **Assuming `as const` freezes at runtime** — `as const` is compile-time only. The object can still be mutated in JavaScript. Use `Object.freeze()` if you need runtime immutability.
2. **Shallow `Readonly`** — `Readonly<T>` only makes top-level properties readonly. Nested objects remain mutable unless you use a deep variant.

## Best Practices
1. **Default to readonly** — Mark function parameters as `readonly` by default. Opt into mutability only when needed.
2. **Use `as const` for configuration objects** — This gives you literal types for string values, which is useful for discriminated unions and exhaustive switches.

## Summary
- `readonly` prevents property reassignment at compile time.
- `ReadonlyArray<T>` disallows array mutation methods like `push` and `splice`.
- `as const` narrows literals and adds deep readonly — but only at compile time.
- For deep immutability, use a recursive `DeepReadonly<T>` utility type.

## Code Examples

**Using as const to derive a string literal union from an array — a common pattern for exhaustive type checking**

```typescript
// as const narrows types and adds readonly
const HTTP_METHODS = ["GET", "POST", "PUT", "DELETE"] as const;
// Type: readonly ["GET", "POST", "PUT", "DELETE"]

// Use as a union type
type HttpMethod = (typeof HTTP_METHODS)[number];
// Type: "GET" | "POST" | "PUT" | "DELETE"

function request(method: HttpMethod, url: string): void {
    // method is narrowed to one of the four literals
}

request("GET", "/api/users");    // OK
// request("PATCH", "/api/users"); // Error: '"PATCH"' is not assignable
```


## Resources

- [TypeScript Handbook: Object Types — readonly](https://www.typescriptlang.org/docs/handbook/2/objects.html#readonly-properties) — Official docs on readonly properties in TypeScript

---

> 📘 *This lesson is part of the [TypeScript Architecture & Patterns](https://stanza.dev/courses/typescript-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*