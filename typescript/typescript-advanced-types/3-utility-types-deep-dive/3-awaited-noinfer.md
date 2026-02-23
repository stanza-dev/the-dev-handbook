---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-awaited-noinfer"
---

# Awaited and NoInfer

## Introduction
TypeScript continues to evolve its set of utility types. `Awaited<T>` (TypeScript 4.5+) recursively unwraps Promise types — solving a problem developers previously handled with custom recursive types. `NoInfer<T>` (TypeScript 5.4+) is a newer addition that prevents TypeScript from inferring a type parameter from a specific position. Both address real pain points in advanced generic programming.

## Key Concepts
- **`Awaited<T>`**: Recursively unwraps `Promise<T>`, handling nested promises and `PromiseLike`. Returns `T` for non-promise types.
- **`NoInfer<T>`**: Blocks type inference at a specific position, forcing TypeScript to infer the type parameter from other usage sites.

## Real World Context
`Awaited` is used everywhere async code meets generics — `Promise.all`, `Promise.race`, and custom async utilities all benefit from it. Before `Awaited`, you had to write your own recursive unwrapping type. `NoInfer` solves a subtle but common problem: when a generic function has multiple positions that could infer a type parameter, and you want to control which position "wins".

## Deep Dive

### Awaited<T>

`Awaited` handles all the edge cases of promise unwrapping:

```typescript
type A = Awaited<Promise<string>>;             // string
type B = Awaited<Promise<Promise<number>>>;    // number (recursively unwrapped)
type C = Awaited<string>;                       // string (non-promise passed through)
type D = Awaited<boolean | Promise<string>>;   // boolean | string
```

The built-in implementation handles `PromiseLike` as well (not just `Promise`), making it compatible with any thenable.

A practical use case is typing `Promise.all`:

```typescript
// Simplified Promise.all typing uses Awaited on each element
async function fetchAll() {
  const [user, posts] = await Promise.all([
    fetchUser(),   // Promise<User>
    fetchPosts(),  // Promise<Post[]>
  ]);
  // user: User, posts: Post[] — Awaited unwrapped both
}
```

### Why Awaited Matters

Before `Awaited`, extracting the resolved type from a `Promise.all` call or a deeply nested async chain required custom types:

```typescript
// Before Awaited (manual approach):
type UnwrapPromise<T> = T extends Promise<infer U>
  ? U extends Promise<any>
    ? UnwrapPromise<U>
    : U
  : T;

// After Awaited (just use the built-in):
type Result = Awaited<Promise<Promise<Promise<string>>>>; // string
```

The built-in version is more robust because it handles `PromiseLike`, unions, and other edge cases.

### NoInfer<T>

`NoInfer` blocks inference at a specific position. Consider this common pattern:

```typescript
// Without NoInfer:
function createSignal<T>(value: T, defaultValue: T): T {
  return value ?? defaultValue;
}

// Problem: TypeScript widens T based on BOTH arguments
const signal = createSignal("hello", 42);
// T is inferred as string | number (from both positions)
```

With `NoInfer`, you can tell TypeScript to only infer `T` from the first argument:

```typescript
// With NoInfer:
function createSignal<T>(value: T, defaultValue: NoInfer<T>): T {
  return value ?? defaultValue;
}

const signal = createSignal("hello", 42);
// Error! Type 'number' is not assignable to type 'string'
// T is inferred as 'string' (only from first argument)
```

Now `T` is inferred solely from `value`, and `defaultValue` must match that inferred type.

### NoInfer for Constrained Defaults

Another common use case is ensuring a default value matches a constrained generic:

```typescript
function createStore<T extends string>(actions: T[], defaultAction: NoInfer<T>): void {
  // ...
}

createStore(["increment", "decrement"], "increment");  // OK
createStore(["increment", "decrement"], "reset");      // Error: "reset" not in the union
```

Without `NoInfer`, TypeScript would widen `T` to include `"reset"`, defeating the purpose of the constraint.

## Common Pitfalls
1. **Using `Awaited` on non-promise types unnecessarily** — `Awaited<string>` is just `string`. It is harmless but adds noise. Only use it when the input might be a `Promise`.
2. **Overusing `NoInfer`** — Only apply `NoInfer` when you have a specific inference problem. Most generic functions infer correctly without it.

## Best Practices
1. **Prefer `Awaited` over custom promise unwrapping** — The built-in handles edge cases (PromiseLike, nested promises, unions) that custom implementations often miss.
2. **Use `NoInfer` for "dependent default" patterns** — When a function has a parameter that should match the inferred type but should not influence inference, wrap it in `NoInfer`.

## Summary
- `Awaited<T>` recursively unwraps promises and handles all thenable types.
- `NoInfer<T>` prevents a position from contributing to type parameter inference.
- `Awaited` eliminates the need for custom recursive promise unwrapping.
- `NoInfer` solves inference conflicts in functions with multiple positions for the same type parameter.

## Code Examples

**Using NoInfer to ensure a default/initial value must be one of the values inferred from another parameter — a common pattern for state machines and configuration**

```typescript
// NoInfer ensures the default must match the inferred allowed values
function createFSM<S extends string>(
  states: S[],
  initial: NoInfer<S>
) {
  return { current: initial, states };
}

// T inferred from states array only
const machine = createFSM(
  ["idle", "loading", "error"],
  "idle" // Must be one of the above — TypeScript checks this
);

createFSM(
  ["idle", "loading", "error"],
  "running" // Error: '"running"' is not assignable to '"idle" | "loading" | "error"'
);
```


## Resources

- [TypeScript 5.4 Release Notes: NoInfer](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-4.html) — Official release notes introducing the NoInfer utility type
- [TypeScript Handbook: Awaited](https://www.typescriptlang.org/docs/handbook/utility-types.html#awaitedtype) — Official documentation for the Awaited utility type

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*