---
source_course: "typescript"
source_lesson: "typescript-unknown-never"
---

# Unknown vs Any

## Introduction
TypeScript has two types for representing values when you do not know the exact type: `any` and `unknown`. While they might seem similar, they are fundamentally different in safety. Understanding the distinction is crucial for writing TypeScript that actually protects you from runtime errors.

## Key Concepts
- **`any` Type**: Disables all type checking for a value. Anything goes — no errors, no safety.
- **`unknown` Type**: The type-safe counterpart of `any`. You can assign any value to `unknown`, but you cannot use it without first narrowing the type.
- **`never` Type**: Represents values that never occur. Used for exhaustiveness checks and functions that never return.

## Real World Context
When parsing JSON from an API, the result is `unknown` (or `any` depending on the function). Using `unknown` forces you to validate the shape before accessing properties, preventing crashes from unexpected data. The `never` type helps catch unhandled cases in switch statements and union exhaustiveness checks.

## Deep Dive

### The Problem with `any`

The `any` type is a complete escape hatch:

```typescript
let data: any = fetchExternalData();
data.name.first.toUpperCase(); // No compile error — crashes at runtime if structure is wrong
data();                        // No compile error — crashes if data is not a function
data[100].nested;              // No compile error — crashes if data is not an array
```

With `any`, TypeScript provides zero protection. Every access is allowed, and bugs hide until runtime.

### The Safety of `unknown`

The `unknown` type forces you to check before using:

```typescript
let data: unknown = fetchExternalData();

// data.name; // Error: 'data' is of type 'unknown'
// data();    // Error: 'data' is of type 'unknown'

if (typeof data === "string") {
  console.log(data.toUpperCase()); // OK — narrowed to string
}

if (data !== null && typeof data === "object" && "name" in data) {
  console.log((data as { name: string }).name); // OK — validated
}
```

The key difference: `unknown` is assignable FROM anything, but assignable TO nothing (except `any` and `unknown`) without narrowing.

### The `never` Type

The `never` type represents impossibility:

```typescript
// A function that never returns
function throwError(message: string): never {
  throw new Error(message);
}

// An impossible intersection
type Impossible = string & number; // never

// Exhaustiveness checking
type Color = "red" | "green" | "blue";

function getHex(color: Color): string {
  switch (color) {
    case "red": return "#ff0000";
    case "green": return "#00ff00";
    case "blue": return "#0000ff";
    default: {
      const exhaustive: never = color;
      return exhaustive; // Compile error if a Color variant is unhandled
    }
  }
}
```

The `never` type has no values, so assigning to it in the default case only succeeds when all possible cases are handled. Adding a new `Color` variant without a matching case produces a compile error.

### Comparison Table

Here is a quick mental model:

```typescript
// any:     opt out of type system entirely
// unknown: safe top type — must narrow before use
// never:   bottom type — no value can have this type
```

`any` is the most permissive, `never` is the most restrictive, and `unknown` sits in between as the safe choice.

## Common Pitfalls
1. **Using `any` when `unknown` would work** — Reaching for `any` is tempting because it silences errors, but `unknown` provides the same flexibility with actual safety. Always prefer `unknown` for values of uncertain type.
2. **Forgetting that `never` means unreachable** — If TypeScript narrows a value to `never`, it means that code path should be impossible. If you see `never` in an unexpected place, check your logic.

## Best Practices
1. **Replace `any` with `unknown` in new code** — Enable the `noImplicitAny` compiler option and use `unknown` for values that need runtime validation.
2. **Use `never` for exhaustiveness** — The default-case-never pattern catches missing switch cases at compile time.

## Summary
- `any` disables type checking entirely and should be avoided in new code.
- `unknown` is the safe alternative that requires narrowing before use.
- `never` represents impossible values and powers exhaustiveness checking.
- Prefer `unknown` over `any` for values of uncertain type.

## Code Examples

**Using unknown with runtime validation to safely parse untyped data — every property is checked before access**

```typescript
// Safely parsing unknown JSON data
function parseConfig(raw: unknown): { port: number; host: string } {
  if (
    raw !== null &&
    typeof raw === "object" &&
    "port" in raw &&
    "host" in raw &&
    typeof (raw as Record<string, unknown>).port === "number" &&
    typeof (raw as Record<string, unknown>).host === "string"
  ) {
    return raw as { port: number; host: string };
  }
  throw new Error("Invalid config format");
}

const config = parseConfig(JSON.parse('{"port": 3000, "host": "localhost"}'));
console.log(config.port); // 3000
```


## Resources

- [TypeScript Handbook: More on Functions (never)](https://www.typescriptlang.org/docs/handbook/2/functions.html#never) — Official documentation on the never type and its uses
- [TypeScript Handbook: Narrowing (The unknown type)](https://www.typescriptlang.org/docs/handbook/2/narrowing.html) — How unknown works with narrowing techniques

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*