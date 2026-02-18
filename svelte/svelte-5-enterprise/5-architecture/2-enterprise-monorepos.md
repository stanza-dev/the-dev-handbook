---
source_course: "svelte-5-enterprise"
source_lesson: "svelte-5-enterprise-monorepos"
---

# Monorepos for Svelte Projects

Share code across multiple apps with a monorepo.

## Monorepo Structure

```
my-monorepo/
├── apps/
│   ├── web/               # Main SvelteKit app
│   │   ├── package.json
│   │   └── src/
│   ├── admin/             # Admin SvelteKit app
│   │   └── package.json
│   └── docs/              # Documentation site
├── packages/
│   ├── ui/                # Shared components
│   │   ├── package.json
│   │   └── src/
│   │       ├── Button.svelte
│   │       └── index.js
│   ├── core/              # Business logic
│   │   ├── package.json
│   │   └── src/
│   │       └── cart.svelte.js
│   └── config/            # Shared configs
│       ├── eslint/
│       └── tsconfig/
├── package.json
├── pnpm-workspace.yaml    # or npm/yarn workspaces
└── turbo.json             # Turborepo config
```

## Workspace Configuration

```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

## Sharing Svelte Components

```json
// packages/ui/package.json
{
  "name": "@myorg/ui",
  "svelte": "./src/index.js",
  "exports": {
    ".": {
      "svelte": "./src/index.js"
    }
  }
}
```

```javascript
// packages/ui/src/index.js
export { default as Button } from './Button.svelte';
export { default as Card } from './Card.svelte';
```

```javascript
// apps/web - Using shared components
import { Button, Card } from '@myorg/ui';
```

## Sharing Runes Logic

```javascript
// packages/core/src/cart.svelte.js
export class Cart {
  items = $state([]);
  
  get total() {
    return this.items.reduce((sum, i) => sum + i.price, 0);
  }
  
  add(item) {
    this.items = [...this.items, item];
  }
}
```

This works in Svelte apps, React apps (with adapter), or Node.js scripts!

## Build Tools

**Turborepo** - Fast builds with caching
```bash
npx create-turbo@latest
```

**Nx** - Full-featured monorepo tool
```bash
npx create-nx-workspace
```

📖 [Turborepo](https://turbo.build/repo)

---

> 📘 *This lesson is part of the [Enterprise Svelte: Testing & Architecture](https://stanza.dev/courses/svelte-5-enterprise) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*