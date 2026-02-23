---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-key-remapping"
---

# Key Remapping

## Introduction
TypeScript 4.1 introduced key remapping in mapped types via the `as` clause. This lets you rename, transform, or filter keys during mapping — a capability that was previously impossible without complex workarounds. Key remapping is how you build types like `Getters<T>`, event handler maps, and filtered property subsets.

## Key Concepts
- **`as` clause**: In `[K in keyof T as NewKey]`, the `as NewKey` expression remaps each key to a new name.
- **Filtering with `never`**: Returning `never` from the `as` clause removes that key entirely.
- **Template literal keys**: Combine `as` with template literals to transform key names (e.g., `as \`get${Capitalize<K>}\``).

## Real World Context
Key remapping is used in production code for generating API client methods from endpoint definitions, creating getter/setter types from state shapes, building event handler types from event name lists, and filtering object types to only include properties of a certain type. It is one of the most practically useful advanced type features.

## Deep Dive

### Basic Key Remapping with `as`

The `as` clause transforms the key during iteration:

```typescript
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number }
```

The `string & K` intersection is needed because `keyof T` could include `symbol` keys, and `Capitalize` only works on strings.

### Filtering Keys with `never`

Returning `never` from the `as` clause removes a key from the output:

```typescript
// Keep only string-valued properties
type OnlyStrings<T> = {
  [K in keyof T as T[K] extends string ? K : never]: T[K];
};

interface Mixed {
  name: string;
  age: number;
  email: string;
  active: boolean;
}

type StringProps = OnlyStrings<Mixed>;
// { name: string; email: string }
```

The conditional `T[K] extends string ? K : never` evaluates for each property. Properties whose values are not strings produce `never` as the key, which removes them from the mapped type.

### Remapping with Exclude

You can use `Exclude` in the `as` clause to remove specific keys:

```typescript
type OmitImplementation<T, K extends keyof T> = {
  [P in keyof T as Exclude<P, K>]: T[P];
};

type WithoutAge = OmitImplementation<Person, "age">;
// { name: string }
```

This is actually how the built-in `Omit<T, K>` could be implemented using key remapping.

### Building a Prefix Map

Key remapping is powerful for generating prefixed or transformed APIs:

```typescript
type PrefixedAPI<T, Prefix extends string> = {
  [K in keyof T as `${Prefix}${Capitalize<string & K>}`]: T[K];
};

interface CRUD {
  create: (data: unknown) => void;
  read: (id: string) => unknown;
  update: (id: string, data: unknown) => void;
  delete: (id: string) => void;
}

type UserAPI = PrefixedAPI<CRUD, "user">;
// { userCreate: ...; userRead: ...; userUpdate: ...; userDelete: ... }
```

This pattern is used in code generators that produce typed client SDKs from API definitions.

## Common Pitfalls
1. **Forgetting `string &` before `Capitalize`** — `Capitalize<K>` fails when `K` might be a `symbol`. Always use `Capitalize<string & K>` to narrow to string keys only.
2. **Expecting key remapping to preserve modifiers** — Key remapping creates a fresh mapped type. If the original properties were `readonly` or optional, you need to explicitly preserve those modifiers.

## Best Practices
1. **Use key remapping for clean API surface types** — Instead of manual interface declarations, derive API types from a base definition using `as` with template literals.
2. **Combine filtering and renaming in one mapped type** — You can both filter (return `never` for unwanted keys) and rename (use template literals) in the same `as` clause.

## Summary
- The `as` clause in mapped types remaps keys to new names.
- Return `never` from `as` to filter out unwanted properties.
- Template literals in `as` enable key name transformations like prefixing and capitalizing.
- Always use `string & K` when applying string manipulation utility types to mapped type keys.

## Code Examples

**A single mapped type that filters to only function-valued properties and renames them with an 'on' prefix — combining key filtering and remapping**

```typescript
// Filter + rename in a single mapped type
type EventEmitter<T> = {
  [K in keyof T as T[K] extends (...args: any[]) => any
    ? `on${Capitalize<string & K>}`
    : never
  ]: T[K];
};

interface AppEvents {
  click: (x: number, y: number) => void;
  title: string; // not a function — will be filtered out
  resize: (w: number, h: number) => void;
  version: number; // not a function — will be filtered out
}

type Emitter = EventEmitter<AppEvents>;
// {
//   onClick: (x: number, y: number) => void;
//   onResize: (w: number, h: number) => void;
// }
```


## Resources

- [TypeScript Handbook: Key Remapping via as](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#key-remapping-via-as) — Official documentation on key remapping in mapped types

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*