---
source_course: "typescript"
source_lesson: "typescript-generics-basics"
---

# Generics: Types as Variables

## Introduction
Generics are one of TypeScript's most powerful features. They let you write functions, interfaces, and classes that work with any type while preserving full type safety. Instead of choosing between `any` (flexible but unsafe) and specific types (safe but rigid), generics give you both flexibility and safety.

## Key Concepts
- **Type Parameter**: A placeholder type (typically `T`) that is determined when the generic is used.
- **Type Inference**: TypeScript automatically determines the type parameter from the arguments you pass.
- **Generic Function**: A function that takes one or more type parameters, making it work with any type.
- **Generic Interface**: An interface with type parameters that create reusable, typed data structures.

## Real World Context
Every time you use `Array<string>`, `Promise<Response>`, or `Map<string, number>`, you are using generics. React's `useState<number>()` is generic. API wrappers, data structures, and utility functions all rely on generics to be both reusable and type-safe.

## Deep Dive

### The Problem Without Generics

Consider a function that returns the first element of an array:

```typescript
// Without generics — loses type information
function firstElement(arr: any[]): any {
  return arr[0];
}

const result = firstElement([1, 2, 3]); // result is 'any' — no type safety
```

The function works for any array, but the return type is `any`. You lose all information about what type was in the array.

### Generics Preserve Type Information

With a type parameter, the type flows through:

```typescript
function firstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

const num = firstElement([1, 2, 3]);        // num is number | undefined
const str = firstElement(["a", "b", "c"]);  // str is string | undefined
const user = firstElement([{ name: "Alice" }]); // user is { name: string } | undefined
```

The `<T>` declares a type parameter. When you call `firstElement([1, 2, 3])`, TypeScript infers `T = number` and the return type becomes `number | undefined`.

### Generic Interfaces

Interfaces can be generic too:

```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
  timestamp: Date;
}

const userResponse: ApiResponse<{ name: string; email: string }> = {
  data: { name: "Alice", email: "alice@example.com" },
  status: 200,
  timestamp: new Date(),
};

const listResponse: ApiResponse<string[]> = {
  data: ["item1", "item2"],
  status: 200,
  timestamp: new Date(),
};
```

The `ApiResponse<T>` interface works for any data type. The `T` is replaced with the actual type when you use the interface.

### Multiple Type Parameters

Functions can have multiple type parameters:

```typescript
function mapEntry<K, V>(key: K, value: V): { key: K; value: V } {
  return { key, value };
}

const entry = mapEntry("name", "Alice"); // { key: string; value: string }
const numEntry = mapEntry(1, true);      // { key: number; value: boolean }
```

Each type parameter is independent and inferred separately.

### Generic Type Aliases

Type aliases work with generics just like interfaces:

```typescript
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

const ok: Result<string> = { success: true, data: "hello" };
const fail: Result<string> = { success: false, error: new Error("oops") };
```

Notice the default type parameter `E = Error`. If you do not specify the error type, it defaults to `Error`.

## Common Pitfalls
1. **Using `any` instead of generics** — If you find yourself writing `any` because a function works with multiple types, that is a sign you need a generic.
2. **Over-specifying type arguments** — TypeScript can infer type arguments in most cases. Writing `firstElement<number>([1, 2, 3])` is usually unnecessary — `firstElement([1, 2, 3])` works fine.

## Best Practices
1. **Let TypeScript infer type arguments** — Only specify them explicitly when inference fails or is ambiguous.
2. **Use descriptive names for complex generics** — `T` is fine for one parameter, but use `TKey`, `TValue`, `TItem` when there are multiple parameters or the meaning is not obvious.

## Summary
- Generics are type parameters that make code reusable without sacrificing type safety.
- TypeScript infers type arguments from usage in most cases.
- Generic interfaces and type aliases create reusable, typed data structures.
- Prefer generics over `any` whenever a function works with multiple types.

## Code Examples

**A generic fetch wrapper that returns a typed API response — the caller specifies the expected data shape**

```typescript
// A generic API wrapper that preserves the response type
interface ApiResponse<T> {
  data: T;
  status: number;
}

async function fetchApi<T>(url: string): Promise<ApiResponse<T>> {
  const response = await fetch(url);
  const data: T = await response.json();
  return { data, status: response.status };
}

// TypeScript knows the exact shape of the response
type User = { id: string; name: string };
const result = await fetchApi<User>("/api/users/1");
console.log(result.data.name); // TypeScript knows .name exists
```


## Resources

- [TypeScript Handbook: Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html) — Official guide to generics in TypeScript

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*