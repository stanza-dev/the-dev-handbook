---
source_course: "react-project-architecture"
source_lesson: "react-project-architecture-component-library"
---

# Building a Component Library

Create reusable components for your organization.

## Library Structure

```
packages/ui/
├── src/
│   ├── components/
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.test.tsx
│   │   │   ├── Button.stories.tsx
│   │   │   └── index.ts
│   │   ├── Input/
│   │   ├── Card/
│   │   └── Modal/
│   ├── hooks/
│   ├── utils/
│   └── index.ts
├── package.json
└── tsconfig.json
```

## Component Pattern

```tsx
// Button/Button.tsx
import { forwardRef } from 'react';
import { cn } from '../../utils/cn';
import type { ButtonProps } from './types';

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant = 'primary', size = 'md', children, ...props }, ref) => {
    return (
      <button
        ref={ref}
        className={cn(
          'btn',
          `btn-${variant}`,
          `btn-${size}`,
          className
        )}
        {...props}
      >
        {children}
      </button>
    );
  }
);

Button.displayName = 'Button';
```

## Type Exports

```tsx
// Button/types.ts
export type ButtonVariant = 'primary' | 'secondary' | 'danger' | 'ghost';
export type ButtonSize = 'sm' | 'md' | 'lg';

export type ButtonProps = {
  variant?: ButtonVariant;
  size?: ButtonSize;
  children: React.ReactNode;
} & React.ButtonHTMLAttributes<HTMLButtonElement>;
```

## Public API

```tsx
// src/index.ts

// Components
export { Button } from './components/Button';
export { Input } from './components/Input';
export { Card } from './components/Card';
export { Modal } from './components/Modal';

// Types
export type { ButtonProps, ButtonVariant } from './components/Button/types';
export type { InputProps } from './components/Input/types';

// Hooks
export { useModal } from './hooks/useModal';

// Utils
export { cn } from './utils/cn';
```

## Package.json

```json
{
  "name": "@company/ui",
  "version": "1.0.0",
  "main": "dist/index.js",
  "module": "dist/index.mjs",
  "types": "dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.js",
      "types": "./dist/index.d.ts"
    },
    "./styles.css": "./dist/styles.css"
  },
  "files": ["dist"],
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts",
    "storybook": "storybook dev -p 6006"
  },
  "peerDependencies": {
    "react": ">=18.0.0",
    "react-dom": ">=18.0.0"
  }
}
```

## Usage in Apps

```tsx
// In an app
import { Button, Input, Card } from '@company/ui';
import '@company/ui/styles.css';

function LoginForm() {
  return (
    <Card>
      <Input label="Email" type="email" />
      <Input label="Password" type="password" />
      <Button variant="primary">Login</Button>
    </Card>
  );
}
```

---

> 📘 *This lesson is part of the [React Project Architecture](https://stanza.dev/courses/react-project-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*