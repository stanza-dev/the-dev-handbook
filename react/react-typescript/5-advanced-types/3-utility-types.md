---
source_course: "react-typescript"
source_lesson: "react-typescript-utility-types"
---

# Utility Types for Props

TypeScript utility types that help with React props.

## ComponentProps

Extract props from any component:

```tsx
import { ComponentProps } from 'react';

// Get props from HTML elements
type ButtonProps = ComponentProps<'button'>;
type InputProps = ComponentProps<'input'>;

// Get props from custom components
type MyButtonProps = ComponentProps<typeof MyButton>;

// Extend native element props
type CustomButtonProps = ComponentProps<'button'> & {
  variant: 'primary' | 'secondary';
};
```

## Partial & Required

```tsx
type User = {
  id: string;
  name: string;
  email: string;
};

// All properties optional
type PartialUser = Partial<User>;
// { id?: string; name?: string; email?: string }

// All properties required
type RequiredUser = Required<PartialUser>;
```

## Pick & Omit

```tsx
type User = {
  id: string;
  name: string;
  email: string;
  role: string;
  createdAt: Date;
};

// Only specific props
type UserPreview = Pick<User, 'id' | 'name'>;
// { id: string; name: string }

// All except specific props
type CreateUser = Omit<User, 'id' | 'createdAt'>;
// { name: string; email: string; role: string }
```

## Exclude for Union Types

```tsx
type Size = 'xs' | 'sm' | 'md' | 'lg' | 'xl';

// Remove specific values
type CommonSize = Exclude<Size, 'xs' | 'xl'>;
// 'sm' | 'md' | 'lg'
```

## Record

```tsx
// Object with specific key-value types
type StatusColors = Record<'success' | 'error' | 'warning', string>;
// { success: string; error: string; warning: string }

const colors: StatusColors = {
  success: 'green',
  error: 'red',
  warning: 'yellow'
};
```

## Combining Utilities

```tsx
import { ComponentProps } from 'react';

// Base button props without 'size' (we'll define our own)
type BaseProps = Omit<ComponentProps<'button'>, 'size'>;

// Our custom props
type CustomProps = {
  size: 'sm' | 'md' | 'lg';
  variant: 'primary' | 'secondary';
  isLoading?: boolean;
};

// Combined props
type ButtonProps = BaseProps & CustomProps;

function Button({ size, variant, isLoading, ...rest }: ButtonProps) {
  return <button {...rest} />;
}
```

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*