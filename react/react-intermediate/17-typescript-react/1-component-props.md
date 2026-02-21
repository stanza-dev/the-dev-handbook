---
source_course: "react-intermediate"
source_lesson: "react-typescript-component-props"
---

# Typing Component Props

## Introduction

TypeScript transforms React development by catching prop-related bugs at compile time. Properly typed props provide autocomplete, documentation, and safety.

## Key Concepts

**Props typing** in React uses TypeScript types (preferred over interfaces per modern conventions) to define the shape of data a component accepts.

```tsx
type ButtonProps = {
  label: string;
  onClick: () => void;
  disabled?: boolean;
};
```

## Real World Context

In production codebases:
- Teams catch prop mismatches during development, not in production
- IDE autocomplete makes components self-documenting
- Refactoring is safer with compile-time checks
- Onboarding is faster when types explain the API

## Deep Dive

### Basic Props Typing

```tsx
type UserCardProps = {
  name: string;
  email: string;
  avatar?: string; // Optional prop
  role: 'admin' | 'user' | 'guest'; // Union type
};

function UserCard({ name, email, avatar, role }: UserCardProps) {
  return (
    <div className="user-card">
      {avatar && <img src={avatar} alt={name} />}
      <h3>{name}</h3>
      <p>{email}</p>
      <span className={`badge-${role}`}>{role}</span>
    </div>
  );
}
```

### Children Prop

Use `ReactNode` for the children prop:

```tsx
type CardProps = {
  title: string;
  children: ReactNode; // Any valid JSX
};

function Card({ title, children }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      {children}
    </div>
  );
}
```

### Extending HTML Element Props

```tsx
import { ComponentProps } from 'react';

// Extend button props
type ButtonProps = ComponentProps<'button'> & {
  variant?: 'primary' | 'secondary';
  isLoading?: boolean;
};

function Button({ variant = 'primary', isLoading, children, ...props }: ButtonProps) {
  return (
    <button
      className={`btn btn-${variant}`}
      disabled={isLoading || props.disabled}
      {...props}
    >
      {isLoading ? 'Loading...' : children}
    </button>
  );
}
```

### Default Props with Destructuring

```tsx
type PaginationProps = {
  totalPages: number;
  currentPage?: number;
  onPageChange: (page: number) => void;
};

function Pagination({
  totalPages,
  currentPage = 1, // Default value
  onPageChange
}: PaginationProps) {
  // ...
}
```

## Common Pitfalls

1. **Using `any` for props**: Defeats the purpose of TypeScript. Always define explicit types.
2. **Overusing interfaces**: Types are more flexible for unions and intersections. Use types unless you need declaration merging.
3. **Forgetting optional markers**: Required props without `?` cause errors when omitted.

## Best Practices

- Use `type` over `interface` for props (per user rules and modern conventions)
- Export prop types for consumers: `export type ButtonProps = {...}`
- Use `ComponentProps<'element'>` to extend native element props
- Provide sensible defaults with destructuring
- Use union types for constrained string values instead of plain `string`
- Add JSDoc comments for complex props

## Summary

Typing props catches bugs early and improves DX. Use TypeScript types, extend native element props when needed, and leverage union types for constrained values.

## Code Examples

**Typed props with unions and extended HTML props**

```tsx
import { ReactNode, ComponentProps } from 'react';

// Basic props with union types
type AlertProps = {
  type: 'success' | 'error' | 'warning' | 'info';
  title: string;
  message: string;
  onDismiss?: () => void;
};

function Alert({ type, title, message, onDismiss }: AlertProps) {
  return (
    <div className={`alert alert-${type}`} role="alert">
      <strong>{title}</strong>
      <p>{message}</p>
      {onDismiss && (
        <button onClick={onDismiss} aria-label="Dismiss">
          ×
        </button>
      )}
    </div>
  );
}

// Extending HTML element props
type InputProps = ComponentProps<'input'> & {
  label: string;
  error?: string;
};

function Input({ label, error, id, ...inputProps }: InputProps) {
  const inputId = id || `input-${label.toLowerCase().replace(/\s/g, '-')}`;
  
  return (
    <div className="form-field">
      <label htmlFor={inputId}>{label}</label>
      <input
        id={inputId}
        aria-invalid={!!error}
        aria-describedby={error ? `${inputId}-error` : undefined}
        {...inputProps}
      />
      {error && (
        <span id={`${inputId}-error`} className="error">
          {error}
        </span>
      )}
    </div>
  );
}

// Usage with full type safety
<Alert type="success" title="Saved" message="Your changes have been saved" />
<Input label="Email" type="email" required placeholder="you@example.com" />
```


## Resources

- [TypeScript with React](https://react.dev/learn/typescript) — Official React TypeScript guide
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/) — Comprehensive TypeScript patterns for React

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*