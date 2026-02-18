---
source_course: "svelte-5-enterprise"
source_lesson: "svelte-5-enterprise-folder-structure"
---

# Organizing Large Svelte Projects

As projects grow, structure becomes crucial.

## Feature-Based Structure

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.svelte
│   │   │   └── SignupForm.svelte
│   │   ├── stores/
│   │   │   └── auth.svelte.js
│   │   └── api/
│   │       └── auth.js
│   ├── cart/
│   │   ├── components/
│   │   ├── stores/
│   │   └── api/
│   └── products/
├── lib/
│   ├── components/         # Shared UI components
│   │   ├── Button.svelte
│   │   └── Modal.svelte
│   ├── utils/              # Utilities
│   │   └── format.js
│   └── stores/             # Global stores
│       └── theme.svelte.js
└── routes/
    └── +layout.svelte
```

## SvelteKit Conventions

```
src/
├── lib/                    # $lib alias
│   ├── components/
│   │   ├── ui/             # Generic UI
│   │   └── domain/         # Business-specific
│   ├── server/             # Server-only code
│   │   └── db.js
│   └── stores/
├── routes/
│   ├── (marketing)/        # Layout group
│   │   ├── +layout.svelte
│   │   └── pricing/
│   ├── (app)/              # App layout group
│   │   ├── +layout.svelte
│   │   └── dashboard/
│   └── api/                # API routes
│       └── users/
│           └── +server.js
├── hooks.server.js
└── app.html
```

## Component Organization

```
lib/components/
├── ui/                     # Primitive UI components
│   ├── Button/
│   │   ├── Button.svelte
│   │   ├── Button.test.js
│   │   └── index.js
│   ├── Input/
│   └── Modal/
├── forms/                  # Form-related
│   ├── TextField.svelte
│   └── Select.svelte
├── layout/                 # Layout components
│   ├── Header.svelte
│   └── Sidebar.svelte
└── domain/                 # Business components
    ├── ProductCard.svelte
    └── UserAvatar.svelte
```

## Barrel Exports

```javascript
// lib/components/ui/index.js
export { default as Button } from './Button/Button.svelte';
export { default as Input } from './Input/Input.svelte';
export { default as Modal } from './Modal/Modal.svelte';

// Usage
import { Button, Input, Modal } from '$lib/components/ui';
```

📖 [SvelteKit project structure](https://svelte.dev/docs/kit/project-structure)

---

> 📘 *This lesson is part of the [Enterprise Svelte: Testing & Architecture](https://stanza.dev/courses/svelte-5-enterprise) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*