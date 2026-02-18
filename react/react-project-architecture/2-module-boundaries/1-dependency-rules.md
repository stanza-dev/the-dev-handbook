---
source_course: "react-project-architecture"
source_lesson: "react-project-architecture-dependency-rules"
---

# Dependency Rules

Define clear rules for what can import what.

## Layered Architecture

```
┌─────────────────────────────────────┐
│              Pages/App              │
├─────────────────────────────────────┤
│              Features               │
├─────────────────────────────────────┤
│          Shared/Common              │
├─────────────────────────────────────┤
│          Core/Foundation            │
└─────────────────────────────────────┘

Rule: Can only import from layers below
```

## Concrete Rules

```
src/
├── app/           # Can import: features, shared, core
├── pages/         # Can import: features, shared, core
├── features/      # Can import: shared, core, other features (API only)
├── shared/        # Can import: core only
└── core/          # Can import: nothing internal
```

## Feature Boundary Rules

```tsx
// ✅ Allowed: Import feature's public API
import { useAuth, LoginForm } from '@/features/auth';

// ❌ Forbidden: Import feature internals
import { authReducer } from '@/features/auth/store/authReducer';
import { validatePassword } from '@/features/auth/utils/validation';
```

## ESLint Boundaries Plugin

```bash
npm install -D eslint-plugin-boundaries
```

```js
// .eslintrc.js
module.exports = {
  plugins: ['boundaries'],
  settings: {
    'boundaries/elements': [
      { type: 'app', pattern: 'app/*' },
      { type: 'pages', pattern: 'pages/*' },
      { type: 'features', pattern: 'features/*' },
      { type: 'shared', pattern: 'shared/*' },
      { type: 'core', pattern: 'core/*' },
    ],
  },
  rules: {
    'boundaries/element-types': [
      'error',
      {
        default: 'disallow',
        rules: [
          { from: 'app', allow: ['features', 'shared', 'core'] },
          { from: 'pages', allow: ['features', 'shared', 'core'] },
          { from: 'features', allow: ['shared', 'core'] },
          { from: 'shared', allow: ['core'] },
        ],
      },
    ],
  },
};
```

## Import Restrictions

```js
// no-restricted-imports
module.exports = {
  rules: {
    'no-restricted-imports': [
      'error',
      {
        patterns: [
          // Forbid deep imports into features
          '@/features/*/*/**',
          // Forbid importing from node_modules internals
          'lodash/*',
        ],
      },
    ],
  },
};
```

## Barrel File Strategy

```tsx
// features/auth/index.ts (barrel file)
// Control what's exposed

export { LoginForm } from './components/LoginForm';
export { useAuth } from './hooks/useAuth';
export type { User } from './types';

// Internal - not exported
// - authReducer
// - validatePassword
// - authApi
```

---

> 📘 *This lesson is part of the [React Project Architecture](https://stanza.dev/courses/react-project-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*