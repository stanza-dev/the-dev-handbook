---
source_course: "react-modern-patterns"
source_lesson: "react-code-organization"
---

# Code Organization

## Introduction

As React applications grow, how you organize files, components, hooks, and utilities has a significant impact on developer productivity. A good structure makes it easy to find code, understand dependencies, and onboard new team members.

## Key Concepts

- **Feature-based structure**: Organizing files by feature or domain rather than by file type (components, hooks, utils).
- **Colocation**: Keeping related files close together. A component's styles, tests, and types live alongside the component file.
- **Barrel exports**: Using index.ts files to re-export from a directory, providing a clean public API.

## Real World Context

A growing team has components in /components, hooks in /hooks, utils in /utils, and types in /types. To add a feature, developers must touch files across four directories and keep imports in sync. Switching to a feature-based structure where each feature has its own directory with components, hooks, and types makes changes self-contained.

## Deep Dive

**Feature-based structure:**

```
src/
  features/
    auth/
      components/
        LoginForm.tsx
        AuthProvider.tsx
      hooks/
        useAuth.ts
      api/
        auth-api.ts
      types.ts
      index.ts          # barrel export
    dashboard/
      components/
        DashboardPage.tsx
        StatsCard.tsx
      hooks/
        useDashboardData.ts
      index.ts
  shared/
    components/
      Button.tsx
      Modal.tsx
    hooks/
      useLocalStorage.ts
    utils/
      format.ts
```

**Colocation principles:**
- Tests next to source: `Button.tsx` and `Button.test.tsx` in the same directory.
- Styles next to components: `Button.tsx` and `Button.module.css` together.
- Types next to usage: Feature-specific types in the feature directory.

**Barrel exports** provide clean public APIs:

```tsx
// features/auth/index.ts
export { AuthProvider } from './components/AuthProvider';
export { useAuth } from './hooks/useAuth';
export type { User, AuthState } from './types';
```

This lets consumers import from the feature without knowing internal structure:

```tsx
import { AuthProvider, useAuth } from '@/features/auth';
```

## Common Pitfalls

1. **Organizing by file type at scale** — Having 50 files in /components makes navigation painful. Group by feature instead.
2. **Deep barrel export chains** — Re-exporting through multiple index files can make it hard to trace where code lives. Keep it to one level.

## Best Practices

1. **Colocate related files** — Keep component, test, and style files together. This makes features self-contained and easy to move or delete.
2. **Separate shared from feature-specific** — Shared utilities used across features go in /shared. Feature-specific code stays in the feature directory.

## Summary

- Feature-based organization scales better than type-based organization.
- Colocate tests, styles, and types with the components they belong to.
- Use barrel exports for clean import paths without exposing internal structure.

## Code Examples

**Feature-based file organization with barrel exports**

```tsx
// Feature-based directory structure
// features/auth/index.ts (barrel export)
export { AuthProvider } from './components/AuthProvider';
export { LoginForm } from './components/LoginForm';
export { useAuth } from './hooks/useAuth';
export type { User, AuthState } from './types';

// Usage in other features
import { useAuth, AuthProvider } from '@/features/auth';
```


## Resources

- [Your First Component](https://react.dev/learn/your-first-component) — Official React guide on component organization

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*