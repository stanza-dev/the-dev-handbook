---
source_course: "typescript-production"
source_lesson: "typescript-production-global-augmentation"
---

# Module & Global Augmentation

## Introduction
Sometimes you need to add types to existing modules or the global scope — extending Express's `Request` interface, adding properties to `Window`, or augmenting a third-party library's types. TypeScript's declaration merging and module augmentation let you do this safely.

## Key Concepts
- **Module Augmentation**: Adding new exports or extending existing types of an external module using `declare module "..."`.
- **Global Augmentation**: Adding types to the global scope using `declare global { ... }` inside a module file.
- **Declaration Merging**: TypeScript's ability to combine multiple declarations of the same name into a single definition.

## Real World Context
Express middleware like `passport` adds a `user` property to `req`. Without augmentation, TypeScript does not know `req.user` exists. Module augmentation lets you declare `req.user: User` so that all your route handlers get type-safe access.

## Deep Dive
### Module Augmentation

Extend a third-party module's types:

```typescript
// types/express.d.ts
import { User } from "../models/user";

declare module "express-serve-static-core" {
    interface Request {
        user?: User;
        sessionId: string;
    }
}
```

Now `req.user` and `req.sessionId` are available in all Express route handlers with full type information.

### Global Augmentation

Add to the global scope from within a module file:

```typescript
// types/global.d.ts
export {}; // Makes this a module (required for declare global)

declare global {
    interface Window {
        analytics: {
            track(event: string, data?: Record<string, unknown>): void;
        };
    }

    // Add a global utility type
    type Nullable<T> = T | null;
}
```

The `export {}` is critical — without it, the file is a script (not a module), and `declare global` is not needed.

### Interface Merging

TypeScript merges multiple interface declarations with the same name:

```typescript
interface Config {
    apiUrl: string;
}

interface Config {
    debug: boolean;
}

// Merged result:
// interface Config { apiUrl: string; debug: boolean; }
```

This works because interfaces are open-ended in TypeScript.

## Common Pitfalls
1. **Missing `export {}` in global augmentation files** — Without it, the file is treated as a script and `declare global` is unnecessary (but confusing). Always include `export {}`.
2. **Augmenting the wrong module name** — Express types live in `express-serve-static-core`, not `express`. Check the source `.d.ts` files to find the correct module name.

## Best Practices
1. **Keep augmentations in a `types/` directory** — Centralize all augmentation files for discoverability.
2. **Reference augmentation files in tsconfig** — Ensure your `include` pattern covers the `types/` directory.

## Summary
- `declare module "..."` extends third-party module types.
- `declare global` adds to the global scope from within a module.
- Always include `export {}` in global augmentation files.
- Interface merging lets you extend interfaces across multiple declarations.

## Code Examples

**Augmenting Express Request to include a typed user property — all route handlers get autocomplete and type checking**

```typescript
// types/express.d.ts — augment Express Request
import { User } from "../models/user";

declare module "express-serve-static-core" {
    interface Request {
        user?: User;
    }
}

// Now in any route handler:
import { Request, Response } from "express";

app.get("/profile", (req: Request, res: Response) => {
    if (req.user) {
        res.json({ name: req.user.name }); // Fully typed!
    }
});
```


## Resources

- [TypeScript Handbook: Declaration Merging](https://www.typescriptlang.org/docs/handbook/declaration-merging.html) — Official docs on how TypeScript merges declarations and supports module augmentation

---

> 📘 *This lesson is part of the [TypeScript in Production](https://stanza.dev/courses/typescript-production) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*