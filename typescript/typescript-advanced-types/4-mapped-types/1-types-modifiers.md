---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-mapped-types-modifiers"
---

# Mapped Types and Modifiers

## Introduction
Mapped types let you create new types by iterating over the keys of an existing type and transforming each property. Combined with modifiers (`readonly`, `?`) and the `+`/`-` operators, they are the mechanism behind `Partial`, `Required`, `Readonly`, and many custom utility types. Understanding mapped types is essential for writing your own type transformations.

## Key Concepts
- **Mapped type syntax**: `{ [K in keyof T]: NewType }` iterates over every key of `T` and produces a new property for each.
- **Modifier operators**: `+readonly` / `-readonly` add or remove the read-only modifier; `+?` / `-?` add or remove optionality.
- **Property value transformation**: The right side of the colon can use `T[K]`, conditional types, or any type expression to transform property types.

## Real World Context
Mapped types power form libraries (making all fields optional for partial updates), ORM type systems (adding `readonly` to model properties), and state management (creating getter/setter pairs from a state shape). React's `ComponentProps` utility and Redux's type helpers are built with mapped types.

## Deep Dive

### Basic Mapped Types

A mapped type iterates over a set of keys using `in`:

```typescript
type OptionsFlags<T> = {
  [K in keyof T]: boolean;
};

interface Features {
  darkMode: () => void;
  notifications: () => void;
}

type FeatureFlags = OptionsFlags<Features>;
// { darkMode: boolean; notifications: boolean }
```

Each property of `Features` is transformed: its value type becomes `boolean` regardless of the original type.

### Preserving the Original Type

To keep the original property types, use indexed access `T[K]`:

```typescript
type Identity<T> = {
  [K in keyof T]: T[K];
};
```

This is an identity mapped type — it produces the same type. It becomes useful when you add modifiers.

### Adding and Removing Modifiers

The `+` and `-` operators control `readonly` and `?` modifiers:

```typescript
// Make all properties readonly (+ is implicit)
type Freeze<T> = {
  +readonly [K in keyof T]: T[K];
};

// Remove readonly from all properties
type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};

// Make all properties optional
type MyPartial<T> = {
  [K in keyof T]+?: T[K];
};

// Make all properties required
type MyRequired<T> = {
  [K in keyof T]-?: T[K];
};
```

The `-` operator is the inverse of `+`. Since `+` is the default, you rarely write it explicitly. The `-` operator is what you use to remove existing modifiers.

### Combining Modifiers

You can apply both modifiers at once:

```typescript
// Make all properties required AND mutable
type Concrete<T> = {
  -readonly [K in keyof T]-?: T[K];
};

interface Config {
  readonly host?: string;
  readonly port?: number;
}

type MutableConfig = Concrete<Config>;
// { host: string; port: number }
```

This strips both `readonly` and `?` from every property in a single mapped type.

### Mapping Over a Custom Key Set

You are not limited to `keyof T`. You can map over any union of string literals:

```typescript
type EventHandlers<Events extends string> = {
  [E in Events]: (event: E) => void;
};

type MouseHandlers = EventHandlers<"click" | "mousedown" | "mouseup">;
// { click: (event: "click") => void; mousedown: ...; mouseup: ... }
```

This is how `Record<K, V>` works under the hood: `{ [P in K]: V }`.

## Common Pitfalls
1. **Forgetting that mapped types create new types** — A mapped type does not modify the original type. It produces a completely new type. If you want to extend an existing type, use intersection (`&`) or `extends` in an interface.
2. **Using `in` instead of `in keyof`** — Writing `[K in T]` when you mean `[K in keyof T]` is a common syntax error. `keyof T` gives you the keys; `T` itself may not be a valid key union.

## Best Practices
1. **Build complex types by composing simple mapped types** — Instead of one complicated mapped type, chain several simple ones: `Readonly<Partial<T>>` is clearer than a single mapped type with multiple transformations.
2. **Use mapped types to enforce exhaustive handling** — A mapped type over a union ensures every member has a corresponding property, catching missing cases at compile time.

## Summary
- Mapped types use `[K in keyof T]` to iterate over properties and create new types.
- The `+`/`-` operators add or remove `readonly` and `?` modifiers.
- You can map over any key union, not just `keyof T`.
- Built-in utility types `Partial`, `Required`, and `Readonly` are all mapped types.

## Code Examples

**A mapped type that generates setter method signatures from a readonly state interface — combining key remapping with Capitalize**

```typescript
// Building a type-safe setter API from a state shape
interface AppState {
  readonly theme: "light" | "dark";
  readonly language: string;
  readonly fontSize: number;
}

// Mapped type that creates setter methods
type Setters<T> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void;
};

type AppSetters = Setters<AppState>;
// {
//   setTheme: (value: "light" | "dark") => void;
//   setLanguage: (value: string) => void;
//   setFontSize: (value: number) => void;
// }
```


## Resources

- [TypeScript Handbook: Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html) — Official documentation on mapped types including modifiers and key remapping

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*