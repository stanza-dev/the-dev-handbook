---
source_course: "react-project-architecture"
source_lesson: "react-project-architecture-path-aliases"
---

# Path Aliases and Imports

Simplify imports with TypeScript paths.

## Configure tsconfig.json

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@/components/*": ["src/components/*"],
      "@/features/*": ["src/features/*"],
      "@/hooks/*": ["src/hooks/*"],
      "@/utils/*": ["src/utils/*"],
      "@/types": ["src/types/index.ts"]
    }
  }
}
```

## Before and After

```tsx
// ❌ Before: Relative imports
import { Button } from '../../../components/Button';
import { useAuth } from '../../hooks/useAuth';
import { formatDate } from '../../../utils/formatters';

// ✅ After: Path aliases
import { Button } from '@/components/Button';
import { useAuth } from '@/features/auth';
import { formatDate } from '@/utils/formatters';
```

## Vite Configuration

```ts
// vite.config.ts
import path from 'path';
import { defineConfig } from 'vite';

export default defineConfig({
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

## Next.js Configuration

```json
// tsconfig.json (Next.js supports this automatically)
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

## Import Order Convention

```tsx
// 1. External libraries
import React, { useState, useEffect } from 'react';
import { useQuery } from '@tanstack/react-query';

// 2. Internal aliases
import { Button } from '@/components/Button';
import { useAuth } from '@/features/auth';

// 3. Relative imports (same feature)
import { ProductCard } from './ProductCard';
import { useProductFilters } from '../hooks/useProductFilters';

// 4. Types
import type { Product } from '@/types';

// 5. Styles
import styles from './ProductList.module.css';
```

## ESLint Import Order

```js
// .eslintrc.js
module.exports = {
  plugins: ['import'],
  rules: {
    'import/order': [
      'error',
      {
        groups: [
          'builtin',
          'external',
          'internal',
          'parent',
          'sibling',
          'index',
          'type',
        ],
        pathGroups: [
          {
            pattern: '@/**',
            group: 'internal',
          },
        ],
        'newlines-between': 'always',
        alphabetize: {
          order: 'asc',
        },
      },
    ],
  },
};
```

---

> 📘 *This lesson is part of the [React Project Architecture](https://stanza.dev/courses/react-project-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*