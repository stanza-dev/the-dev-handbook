---
source_course: "react-project-architecture"
source_lesson: "react-project-architecture-monorepo-basics"
---

# Monorepo Fundamentals

Multiple packages in a single repository.

## Why Monorepo?

**Benefits**:
- Shared code without publishing
- Atomic changes across packages
- Unified tooling and config
- Easier refactoring

**Challenges**:
- More complex setup
- Larger repository
- Build coordination needed

## Structure Example

```
my-monorepo/
├── apps/
│   ├── web/              # Main web app
│   ├── admin/            # Admin dashboard
│   └── mobile/           # React Native app
├── packages/
│   ├── ui/               # Shared components
│   ├── utils/            # Shared utilities
│   └── api-client/       # API client
├── package.json
├── nx.json               # Nx config
└── tsconfig.base.json
```

## Package Manager Workspaces

### pnpm (Recommended)

```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

### npm/yarn

```json
// package.json
{
  "workspaces": [
    "apps/*",
    "packages/*"
  ]
}
```

## Internal Package

```json
// packages/ui/package.json
{
  "name": "@myorg/ui",
  "version": "0.0.0",
  "main": "src/index.ts",
  "types": "src/index.ts"
}
```

## Consuming Internal Packages

```json
// apps/web/package.json
{
  "dependencies": {
    "@myorg/ui": "workspace:*",
    "@myorg/utils": "workspace:*"
  }
}
```

```tsx
// apps/web/src/App.tsx
import { Button } from '@myorg/ui';
import { formatDate } from '@myorg/utils';
```

## Shared Configuration

```json
// tsconfig.base.json
{
  "compilerOptions": {
    "strict": true,
    "esModuleInterop": true,
    "jsx": "react-jsx",
    "baseUrl": ".",
    "paths": {
      "@myorg/*": ["packages/*/src"]
    }
  }
}

// apps/web/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "dist"
  },
  "include": ["src"]
}
```

---

> 📘 *This lesson is part of the [React Project Architecture](https://stanza.dev/courses/react-project-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*