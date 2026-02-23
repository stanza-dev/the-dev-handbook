---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-extract-exclude-nonnullable"
---

# Extract, Exclude, and NonNullable

## Introduction
While `Pick` and `Omit` operate on object properties, `Extract`, `Exclude`, and `NonNullable` operate on union members. These distributive conditional types let you filter unions by including or removing specific types. Combined with `ReturnType` and `Parameters`, they form a complete toolkit for type-level data manipulation.

## Key Concepts
- **`Exclude<T, U>`**: Removes from `T` all union members that are assignable to `U`.
- **`Extract<T, U>`**: Keeps from `T` only the union members that are assignable to `U`.
- **`NonNullable<T>`**: Shorthand for `Exclude<T, null | undefined>`.
- **`ReturnType<T>`**: Extracts the return type of a function type.
- **`Parameters<T>`**: Extracts the parameter types of a function type as a tuple.

## Real World Context
These types are essential for discriminated union patterns. When you have a union of event types or action types, `Extract` lets you narrow to a specific variant. `Exclude` lets you handle "everything except" patterns. `ReturnType` and `Parameters` are used in testing (mocking function signatures), middleware (wrapping functions), and code generation.

## Deep Dive

### Exclude<T, U>

`Exclude` removes union members assignable to `U`:

```typescript
type T = string | number | boolean;

type WithoutBooleans = Exclude<T, boolean>; // string | number
type WithoutPrimitives = Exclude<T, string | number>; // boolean
```

This is implemented as `T extends U ? never : T`, which distributes over the union and returns `never` for excluded members.

### Extract<T, U>

`Extract` is the inverse — it keeps only members assignable to `U`:

```typescript
type Events = "click" | "scroll" | "mousemove" | "keydown";
type MouseEvents = Extract<Events, "click" | "scroll" | "mousemove">;
// "click" | "scroll" | "mousemove"

// More commonly, extract by structural type:
type Mixed = string | number | (() => void) | { id: number };
type FunctionTypes = Extract<Mixed, Function>; // () => void
```

### Working with Discriminated Unions

`Extract` shines with discriminated unions:

```typescript
type Action =
  | { type: "ADD_TODO"; payload: string }
  | { type: "REMOVE_TODO"; payload: number }
  | { type: "TOGGLE_TODO"; payload: number };

type AddAction = Extract<Action, { type: "ADD_TODO" }>;
// { type: "ADD_TODO"; payload: string }

type PayloadOf<T extends Action["type"]> =
  Extract<Action, { type: T }>["payload"];

type AddPayload = PayloadOf<"ADD_TODO">; // string
type RemovePayload = PayloadOf<"REMOVE_TODO">; // number
```

This pattern extracts the payload type for any given action type, which is invaluable for type-safe Redux-like patterns.

### ReturnType<T> and Parameters<T>

These built-in types use `infer` to extract parts of function types:

```typescript
function createUser(name: string, age: number) {
  return { id: Math.random(), name, age };
}

type UserReturn = ReturnType<typeof createUser>;
// { id: number; name: string; age: number }

type UserParams = Parameters<typeof createUser>;
// [name: string, age: number]

// Access individual parameters:
type FirstParam = Parameters<typeof createUser>[0]; // string
type SecondParam = Parameters<typeof createUser>[1]; // number
```

These are particularly useful when wrapping or mocking functions where you want the wrapper to have the same signature.

### InstanceType<T> and ConstructorParameters<T>

For class types, there are corresponding utilities:

```typescript
class Service {
  constructor(public name: string, public timeout: number) {}
}

type ServiceInstance = InstanceType<typeof Service>; // Service
type ServiceArgs = ConstructorParameters<typeof Service>; // [name: string, timeout: number]
```

Note that you pass `typeof Service` (the constructor function type), not `Service` (the instance type).

## Common Pitfalls
1. **Confusing `Exclude`/`Extract` with `Omit`/`Pick`** — `Exclude` and `Extract` filter union members. `Omit` and `Pick` filter object properties. They operate on different things.
2. **Passing an instance type to `ReturnType`** — `ReturnType<Service>` is an error. You need `ReturnType<typeof someFunction>` because `ReturnType` expects a function type.

## Best Practices
1. **Use `Extract` for discriminated union narrowing** — Instead of manually writing conditional types, `Extract<Union, { type: "X" }>` cleanly selects a specific variant.
2. **Combine `ReturnType` with `typeof`** — The pattern `ReturnType<typeof myFunction>` derives the return type from a function value without having to export or name the return type separately.

## Summary
- `Exclude<T, U>` removes union members; `Extract<T, U>` keeps them.
- `NonNullable<T>` is `Exclude<T, null | undefined>`.
- `ReturnType<T>` and `Parameters<T>` extract function signatures.
- Use `Extract` with discriminated unions for type-safe pattern matching.
- Always pass function types (not instance types) to `ReturnType` and `Parameters`.

## Code Examples

**Using Extract and Exclude with discriminated unions to build a type-safe event system where each handler receives the correct event shape**

```typescript
// Type-safe event handler pattern using Extract
type AppEvent =
  | { kind: "user:login"; userId: string }
  | { kind: "user:logout"; userId: string }
  | { kind: "page:view"; url: string }
  | { kind: "error"; code: number; message: string };

// Extract specific event types
type UserEvent = Extract<AppEvent, { kind: `user:${string}` }>;
// { kind: "user:login"; userId: string } | { kind: "user:logout"; userId: string }

type NonErrorEvent = Exclude<AppEvent, { kind: "error" }>;
// All events except the error event

// Type-safe handler registry
type EventHandler<K extends AppEvent["kind"]> = (
  event: Extract<AppEvent, { kind: K }>
) => void;

const onLogin: EventHandler<"user:login"> = (event) => {
  console.log(event.userId); // Correctly typed as string
};
```


## Resources

- [TypeScript Handbook: Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html) — Official reference covering Exclude, Extract, ReturnType, Parameters, and more

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*