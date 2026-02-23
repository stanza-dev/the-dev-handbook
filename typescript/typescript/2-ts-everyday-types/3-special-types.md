---
source_course: "typescript"
source_lesson: "typescript-tuples-special-types"
---

# Tuples and Special Types

## Introduction
Beyond arrays and primitives, TypeScript offers tuples for fixed-length typed arrays and several special types for edge cases. Understanding these types rounds out your everyday TypeScript vocabulary and prevents subtle bugs.

## Key Concepts
- **Tuple**: A fixed-length array where each position has a specific type.
- **Literal Type**: A type that represents a single, exact value (e.g., `"admin"` or `42`).
- **`bigint`**: A numeric type for integers larger than `Number.MAX_SAFE_INTEGER`.
- **`symbol`**: A type for unique identifiers created with `Symbol()`.

## Real World Context
Tuples are used throughout TypeScript libraries. React's `useState` returns a tuple: `[state, setState]`. Database query results often come as tuples of `[rows, metadata]`. Literal types power union-based state machines like `"loading" | "success" | "error"`.

## Deep Dive

### Tuples

A tuple has a fixed number of elements with known types at each position:

```typescript
const coordinate: [number, number] = [40.7128, -74.0060];
const userEntry: [string, number, boolean] = ["Alice", 30, true];
```

Unlike arrays, accessing an element beyond the defined length is a compile error:

```typescript
const pair: [string, number] = ["age", 25];
const first = pair[0];  // string
const second = pair[1]; // number
// pair[2]; // Error: Tuple type '[string, number]' has no element at index '2'
```

Tuples can also have optional elements and rest elements:

```typescript
type HttpResponse = [number, string, string?];
const success: HttpResponse = [200, "OK"];
const redirect: HttpResponse = [301, "Moved", "/new-location"];
```

The third element is optional, indicated by the `?`.

### Literal Types

Literal types narrow a type to one specific value:

```typescript
type Direction = "north" | "south" | "east" | "west";
type HttpStatus = 200 | 301 | 404 | 500;

function move(direction: Direction): void {
  console.log(`Moving ${direction}`);
}

move("north"); // OK
// move("up"); // Error: Argument of type '"up"' is not assignable to type 'Direction'
```

Literal types combined with unions create powerful, self-documenting APIs.

### bigint and symbol

The `bigint` type handles integers beyond the safe integer limit:

```typescript
const largeId: bigint = 9007199254740992n;
const result = largeId + 1n; // bigint arithmetic uses the 'n' suffix
// largeId + 1; // Error: cannot mix bigint and number
```

The `symbol` type creates unique identifiers:

```typescript
const uniqueKey: symbol = Symbol("cacheKey");
const anotherKey: symbol = Symbol("cacheKey");
// uniqueKey === anotherKey; // false — every Symbol is unique
```

Symbols are often used as object keys to avoid property name collisions.

## Common Pitfalls
1. **Confusing tuples with arrays** — `[string, number]` is a tuple with exactly two elements. `(string | number)[]` is an array of any length containing strings or numbers. They are fundamentally different.
2. **Mixing bigint and number** — TypeScript does not allow mixing these types in arithmetic. Use consistent types or explicit conversion.

## Best Practices
1. **Use tuples for fixed structures** — When a function returns multiple values (like React hooks), type the return as a tuple to give each position a meaningful type.
2. **Prefer literal unions over enums for simple cases** — `type Status = "active" | "inactive"` is often simpler and more ergonomic than an enum.

## Summary
- Tuples are fixed-length arrays with specific types at each position.
- Literal types restrict values to exact strings, numbers, or booleans.
- `bigint` and `symbol` handle special cases for large numbers and unique identifiers.

## Code Examples

**A function returning a typed tuple with a value and a function — a common pattern in functional APIs**

```typescript
// Tuple return type — models a typed pair of [value, updater]
// (React's useState uses a similar pattern with internal state management)
function createCounter(initial: number): [number, (n: number) => number] {
  const increment = (n: number) => initial + n;
  return [initial, increment];
}

const [count, increment] = createCounter(0);
// count is number, increment is (n: number) => number
console.log(increment(5)); // 5
```


## Resources

- [TypeScript Handbook: Object Types (Tuple Types)](https://www.typescriptlang.org/docs/handbook/2/objects.html#tuple-types) — Official documentation on tuple types and their advanced features

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*