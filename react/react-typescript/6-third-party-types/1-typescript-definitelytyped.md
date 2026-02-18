---
source_course: "react-typescript"
source_lesson: "react-typescript-definitelytyped"
---

# DefinitelyTyped & @types

Many JavaScript libraries have community-maintained type definitions.

## Installing Type Definitions

```bash
# React types (usually pre-installed)
npm install --save-dev @types/react @types/react-dom

# Other libraries
npm install --save-dev @types/lodash
npm install --save-dev @types/node
```

## Libraries with Built-in Types

Some libraries ship their own types:

```bash
# These include types in the main package
npm install @tanstack/react-query  # Built-in types
npm install axios                   # Built-in types
npm install date-fns               # Built-in types
npm install zod                    # Built-in types
```

## Checking for Available Types

Search on npm or DefinitelyTyped:
- https://www.npmjs.com/~types
- https://github.com/DefinitelyTyped/DefinitelyTyped

## When Types Don't Exist

Create a declaration file:

```tsx
// types/untyped-library.d.ts
declare module 'untyped-library' {
  export function doSomething(value: string): number;
  export const VERSION: string;
}
```

## Augmenting Existing Types

```tsx
// types/styled.d.ts
import 'styled-components';

declare module 'styled-components' {
  export interface DefaultTheme {
    colors: {
      primary: string;
      secondary: string;
    };
    spacing: (n: number) => string;
  }
}
```

## Type-Only Imports

```tsx
// Import only types (removed at compile time)
import type { User } from './types';
import type { ComponentProps } from 'react';

// Or inline
import { type User, fetchUser } from './api';
```

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*