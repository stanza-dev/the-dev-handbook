---
source_course: "typescript-advanced-types"
source_lesson: "typescript-advanced-types-recursive-types"
---

# Recursive Types

## Introduction
Recursive types are types that reference themselves in their definition. They are essential for modeling data structures that are inherently recursive: trees, linked lists, nested JSON, file system hierarchies, and deeply nested API responses. TypeScript supports recursive type aliases, enabling you to express these structures with full type safety.

## Key Concepts
- **Recursive type alias**: A type alias that references itself, e.g., `type Tree<T> = { value: T; children: Tree<T>[] }`.
- **Base case**: Every recursive type needs a non-recursive branch to terminate, usually through a union with a primitive type or an empty case.
- **Depth limits**: TypeScript enforces recursion depth limits to prevent infinite type expansion.

## Real World Context
Recursive types model real data everywhere: JSON payloads (which can nest arbitrarily), DOM trees, routing configurations with nested child routes, menu systems with submenus, comment threads with replies, and organizational hierarchies. Libraries like Zod, io-ts, and React Router all use recursive types internally.

## Deep Dive

### Basic Recursive Types

The simplest recursive type is a JSON value:

```typescript
type Json =
  | string
  | number
  | boolean
  | null
  | { [key: string]: Json }
  | Json[];
```

This type allows any valid JSON value: primitives, objects with JSON values, or arrays of JSON values. The recursion occurs in the object and array branches.

### Tree Structures

Trees are the classic recursive data structure:

```typescript
type TreeNode<T> = {
  value: T;
  children: TreeNode<T>[];
};

const fileSystem: TreeNode<string> = {
  value: "root",
  children: [
    {
      value: "src",
      children: [
        { value: "index.ts", children: [] },
        { value: "utils.ts", children: [] },
      ],
    },
    { value: "package.json", children: [] },
  ],
};
```

Leaf nodes have an empty `children` array, serving as the base case.

### Linked Lists

A linked list uses recursion through an optional `next` pointer:

```typescript
type LinkedList<T> = {
  value: T;
  next: LinkedList<T> | null;
};

const list: LinkedList<number> = {
  value: 1,
  next: {
    value: 2,
    next: {
      value: 3,
      next: null, // base case: end of list
    },
  },
};
```

The `| null` in the `next` field is the base case that terminates the recursion.

### Recursive Types with Generics

Recursive types work well with generics for flexible data structures:

```typescript
// A nested route configuration (like React Router)
type Route = {
  path: string;
  component: string;
  children?: Route[];
};

const routes: Route[] = [
  {
    path: "/",
    component: "Layout",
    children: [
      { path: "home", component: "Home" },
      {
        path: "settings",
        component: "Settings",
        children: [
          { path: "profile", component: "Profile" },
          { path: "security", component: "Security" },
        ],
      },
    ],
  },
];
```

The optional `children` array references `Route` again, allowing arbitrary nesting depth.

### Mutually Recursive Types

Two types can reference each other:

```typescript
type Expression =
  | NumberLiteral
  | BinaryExpression;

type NumberLiteral = {
  kind: "number";
  value: number;
};

type BinaryExpression = {
  kind: "binary";
  operator: "+" | "-" | "*" | "/";
  left: Expression;  // references Expression
  right: Expression; // references Expression
};
```

This pattern is common for AST (Abstract Syntax Tree) definitions where different node types reference each other.

## Common Pitfalls
1. **Forgetting the base case** — A recursive type without a termination condition (like `| null` or `| never`) will be valid to define but impossible to construct a finite value for. Always include a non-recursive branch.
2. **Hitting recursion depth limits** — TypeScript limits type instantiation depth to prevent infinite expansion. If your recursive type is too deep, consider restructuring or using `any` at the deepest levels.

## Best Practices
1. **Use optional properties for optional recursion** — `children?: TreeNode<T>[]` is cleaner than `children: TreeNode<T>[] | undefined` and signals that the recursion is optional.
2. **Prefer discriminated unions for recursive type variants** — Use a `kind` or `type` discriminant field to distinguish between recursive and non-recursive branches, enabling type narrowing.

## Summary
- Recursive types reference themselves in their definition.
- They model trees, linked lists, JSON, ASTs, and nested configurations.
- Every recursive type needs a base case to terminate.
- TypeScript enforces depth limits on recursive type instantiation.

## Code Examples

**A recursive form field type that supports nested field groups — a pattern used in dynamic form builders like React Hook Form and Formik**

```typescript
// A recursive type for deeply nested form state
type FormField = {
  name: string;
  type: "text" | "number" | "email" | "group";
  value?: string | number;
  validation?: {
    required?: boolean;
    min?: number;
    max?: number;
  };
  // Recursive: groups contain child fields
  children?: FormField[];
};

const registrationForm: FormField = {
  name: "registration",
  type: "group",
  children: [
    { name: "username", type: "text", validation: { required: true } },
    { name: "email", type: "email", validation: { required: true } },
    {
      name: "address",
      type: "group",
      children: [
        { name: "street", type: "text" },
        { name: "zip", type: "number", validation: { min: 10000, max: 99999 } },
      ],
    },
  ],
};
```


## Resources

- [TypeScript Handbook: Recursive Type Aliases](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#more-recursive-type-aliases) — TypeScript 3.7 release notes on recursive type alias support

---

> 📘 *This lesson is part of the [TypeScript Type System Mastery](https://stanza.dev/courses/typescript-advanced-types) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*