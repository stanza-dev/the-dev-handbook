---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-mapped-types-modifiers"
---

# Mapped Types

Mapped types allow you to create new types by iterating over keys of an existing type.

```typescript
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};
```

## Mapping Modifiers

You can add or remove `readonly` and `?` modifiers using `+` (default) or `-`.

```typescript
// Removes 'readonly' attributes from a type's properties
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};

// Removes optional attributes (makes them required)
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};
```

## Key Remapping

You can change the key name using `as`.

```typescript
type Getters<Type> = {
    [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};
```

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*