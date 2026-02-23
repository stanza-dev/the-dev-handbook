---
source_course: "typescript"
source_lesson: "typescript-declaration-merging"
---

# Declaration Merging

## Introduction
Declaration merging is a unique TypeScript feature where the compiler combines multiple declarations with the same name into a single definition. This is one of the key capabilities that sets interfaces apart from type aliases and is essential for augmenting third-party types.

## Key Concepts
- **Declaration Merging**: When two or more declarations share the same name, TypeScript merges their members into one definition.
- **Interface Merging**: The most common form — multiple `interface` declarations with the same name are automatically combined.
- **Module Augmentation**: Extending types from external packages by declaring additional interface members.

## Real World Context
When you add a custom property to Express's `Request` object (like `req.user` after authentication middleware), you use declaration merging. Libraries like Prisma, Express, and Next.js rely on this pattern to let consumers extend their types without modifying source code.

## Deep Dive

### Interface Merging

When you declare an interface with the same name twice, TypeScript merges them:

```typescript
interface Settings {
  theme: "light" | "dark";
  language: string;
}

interface Settings {
  notifications: boolean;
  fontSize: number;
}

// The merged interface has all four properties
const userSettings: Settings = {
  theme: "dark",
  language: "en",
  notifications: true,
  fontSize: 14,
};
```

Both declarations contribute their properties to a single `Settings` type. This works across files as long as the declarations are in the same scope.

### Why Type Aliases Cannot Merge

Type aliases do not support merging. Declaring the same type name twice is an error:

```typescript
type Config = { debug: boolean };
// type Config = { verbose: boolean }; // Error: Duplicate identifier 'Config'
```

This is a fundamental difference: interfaces are open (can be extended), while type aliases are closed (defined once).

### Module Augmentation

The most practical use of declaration merging is extending third-party types:

```typescript
// Augmenting Express Request to include a user property
import { Request } from "express";

declare module "express" {
  interface Request {
    user?: {
      id: string;
      role: string;
    };
  }
}

// Now req.user is recognized everywhere
function authMiddleware(req: Request): void {
  req.user = { id: "u123", role: "admin" };
}
```

The `declare module` syntax opens the module's type scope, and the interface declaration merges with the existing `Request` interface.

### Global Augmentation

You can also extend global types:

```typescript
declare global {
  interface Window {
    analytics: {
      track: (event: string, data?: Record<string, unknown>) => void;
    };
  }
}

// Now window.analytics.track is recognized
window.analytics.track("page_view", { page: "/home" });
```

This is how analytics libraries, feature flags, and custom globals are typed.

## Common Pitfalls
1. **Accidental merging** — If you create an interface with the same name as an existing one in the same scope, they merge silently. This can introduce unexpected properties. Use unique names or namespaces to avoid collisions.
2. **Expecting type aliases to merge** — Attempting to redeclare a type alias produces an error. If you need merging, use an interface.

## Best Practices
1. **Use module augmentation for extending libraries** — Never modify a library's type definitions directly. Use `declare module` to add properties safely.
2. **Keep merged declarations close together** — If an interface is split across files, document where the merging happens to help other developers find all members.

## Summary
- Declaration merging automatically combines multiple interface declarations with the same name.
- Type aliases cannot merge — they produce duplicate identifier errors.
- Module augmentation uses merging to extend third-party types safely.

## Code Examples

**Module augmentation adds custom properties to Express's Request interface without modifying the library's source**

```typescript
// Extending a third-party library type via module augmentation
declare module "express" {
  interface Request {
    sessionId?: string;
    user?: { id: string; email: string };
  }
}

// After augmentation, these properties are recognized on all Request objects
// req.sessionId — string | undefined
// req.user     — { id: string; email: string } | undefined
```


## Resources

- [TypeScript Handbook: Declaration Merging](https://www.typescriptlang.org/docs/handbook/declaration-merging.html) — Official guide to declaration merging for interfaces, namespaces, and modules

---

> 📘 *This lesson is part of the [TypeScript Essentials](https://stanza.dev/courses/typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*