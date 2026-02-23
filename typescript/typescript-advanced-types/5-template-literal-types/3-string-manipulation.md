---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-string-manipulation"
---

# Intrinsic String Types

## Introduction
TypeScript provides four built-in string manipulation types that transform string literal types at the type level: `Uppercase`, `Lowercase`, `Capitalize`, and `Uncapitalize`. These are called "intrinsic" types because they are implemented directly in the compiler rather than as user-defined conditional types. Combined with template literals, they form a complete toolkit for type-level string processing.

## Key Concepts
- **`Uppercase<S>`**: Converts every character to uppercase: `"hello"` becomes `"HELLO"`.
- **`Lowercase<S>`**: Converts every character to lowercase: `"HELLO"` becomes `"hello"`.
- **`Capitalize<S>`**: Uppercases only the first character: `"hello"` becomes `"Hello"`.
- **`Uncapitalize<S>`**: Lowercases only the first character: `"Hello"` becomes `"hello"`.

## Real World Context
These types are essential for generating type-safe API interfaces from schema definitions. CSS-in-JS libraries use them to type camelCase property names from kebab-case CSS properties. Event systems use `Capitalize` to generate handler names. Code generators use `Uncapitalize` to derive variable names from class names.

## Deep Dive

### The Four Intrinsic Types

Each type operates on string literal types:

```typescript
type A = Uppercase<"hello">;      // "HELLO"
type B = Lowercase<"HELLO">;      // "hello"
type C = Capitalize<"hello">;     // "Hello"
type D = Uncapitalize<"Hello">;   // "hello"
```

They also distribute over unions:

```typescript
type Methods = "get" | "post" | "put" | "delete";
type UpperMethods = Uppercase<Methods>;
// "GET" | "POST" | "PUT" | "DELETE"
```

### Combining with Template Literals

The most common use is within template literal types:

```typescript
type Events = "click" | "focus" | "blur";

// Generate handler names
type OnHandler = `on${Capitalize<Events>}`;
// "onClick" | "onFocus" | "onBlur"

// Generate constant names
type EventConst = `EVENT_${Uppercase<Events>}`;
// "EVENT_CLICK" | "EVENT_FOCUS" | "EVENT_BLUR"

// Generate CSS custom property names
type CSSVar = `--${Lowercase<Events>}-color`;
// "--click-color" | "--focus-color" | "--blur-color"
```

These patterns generate exhaustive string literal unions that prevent typos.

### Pattern Matching and Extraction

Combine intrinsic types with `infer` to parse and transform strings:

```typescript
// Convert camelCase to snake_case (simplified for one word boundary)
type CamelToSnake<S extends string> =
  S extends `${infer Head}${infer Tail}`
    ? Tail extends Capitalize<Tail>
      ? `${Lowercase<Head>}_${CamelToSnake<Uncapitalize<Tail>>}`
      : `${Lowercase<Head>}${CamelToSnake<Tail>}`
    : S;
```

While complex recursive string manipulation is possible, it is often better kept simple. The main practical use cases are:

```typescript
// Extract prefix and transform
type RemovePrefix<S extends string, P extends string> =
  S extends `${P}${infer Rest}` ? Uncapitalize<Rest> : S;

type A = RemovePrefix<"onClick", "on">; // "click"
type B = RemovePrefix<"onMouseDown", "on">; // "mouseDown"
```

### Building a Type-Safe Event System

Here is a complete example combining all four intrinsic types:

```typescript
type DOMEvent = "click" | "mouseDown" | "keyPress" | "scroll";

// Handler names: onClick, onMouseDown, etc.
type HandlerName<E extends string> = `on${Capitalize<E>}`;

// Constant names: CLICK, MOUSE_DOWN, etc. (simplified)
type ConstantName<E extends string> = Uppercase<E>;

// CSS class names: event-click, event-mouse-down, etc. (simplified)
type ClassName<E extends string> = `event-${Lowercase<E>}`;

type ClickHandler = HandlerName<"click">;  // "onClick"
type ClickConst = ConstantName<"click">;   // "CLICK"
type ClickClass = ClassName<"click">;      // "event-click"
```

Each intrinsic type handles a different naming convention from the same source.

## Common Pitfalls
1. **Applying intrinsic types to `string` (wide type)** — `Uppercase<string>` returns `string`, not a useful narrowed type. Intrinsic types are most useful with string literal types or unions of literals.
2. **Confusing `Capitalize` with `Uppercase`** — `Capitalize` only changes the first character; `Uppercase` changes all characters. `Capitalize<"hello">` is `"Hello"`, not `"HELLO"`.

## Best Practices
1. **Use `Capitalize` with `on` prefix for event handler patterns** — The `` `on${Capitalize<EventName>}` `` pattern is the standard way to generate handler names in TypeScript.
2. **Use `Uppercase` for enum-like constant naming** — When deriving constant names from a union of lowercase values, `Uppercase` produces conventional SCREAMING_CASE identifiers.

## Summary
- `Uppercase`, `Lowercase`, `Capitalize`, and `Uncapitalize` are compiler-intrinsic string manipulation types.
- They distribute over unions automatically.
- Combine them with template literals for type-safe string generation.
- `Capitalize` + template prefix is the standard event handler naming pattern.

## Code Examples

**Generating a fully typed API client interface by combining CRUD actions with resource names using Capitalize and template literal distribution**

```typescript
// Building a type-safe API client from method definitions
type CRUDAction = "create" | "read" | "update" | "delete";
type Resource = "user" | "post";

// Generate method names: createUser, readPost, etc.
type APIMethod = `${CRUDAction}${Capitalize<Resource>}`;
// "createUser" | "createPost" | "readUser" | "readPost" |
// "updateUser" | "updatePost" | "deleteUser" | "deletePost"

// Generate a typed API client
type APIClient = {
  [M in APIMethod]: () => Promise<unknown>;
};

// Implementation can be checked:
const client: APIClient = {
  createUser: async () => {},
  createPost: async () => {},
  readUser: async () => {},
  readPost: async () => {},
  updateUser: async () => {},
  updatePost: async () => {},
  deleteUser: async () => {},
  deletePost: async () => {},
  // Missing or misspelled methods would be compile errors
};
```


## Resources

- [TypeScript Handbook: Intrinsic String Manipulation Types](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html#intrinsic-string-manipulation-types) — Official documentation on Uppercase, Lowercase, Capitalize, and Uncapitalize

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*