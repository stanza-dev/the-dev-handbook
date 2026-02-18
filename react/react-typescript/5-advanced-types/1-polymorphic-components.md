---
source_course: "react-typescript"
source_lesson: "react-typescript-polymorphic-components"
---

# Polymorphic Components

Components that can render as different HTML elements.

## The 'as' Prop Pattern

```tsx
import { ComponentPropsWithoutRef, ElementType, ReactNode } from 'react';

// Generic type for polymorphic component
type BoxProps<E extends ElementType> = {
  as?: E;
  children: ReactNode;
} & ComponentPropsWithoutRef<E>;

function Box<E extends ElementType = 'div'>({
  as,
  children,
  ...props
}: BoxProps<E>) {
  const Component = as || 'div';
  return <Component {...props}>{children}</Component>;
}

// Usage
<Box>Default div</Box>
<Box as="span">As span</Box>
<Box as="a" href="/home">As link</Box>
<Box as="button" onClick={() => {}}>As button</Box>
```

## With Custom Props

```tsx
type TextProps<E extends ElementType> = {
  as?: E;
  size?: 'sm' | 'md' | 'lg';
  weight?: 'normal' | 'bold';
  children: ReactNode;
} & Omit<ComponentPropsWithoutRef<E>, 'size'>;

function Text<E extends ElementType = 'span'>({
  as,
  size = 'md',
  weight = 'normal',
  children,
  ...props
}: TextProps<E>) {
  const Component = as || 'span';
  
  return (
    <Component
      className={`text-${size} font-${weight}`}
      {...props}
    >
      {children}
    </Component>
  );
}

// Usage
<Text>Default span</Text>
<Text as="h1" size="lg" weight="bold">Heading</Text>
<Text as="p" size="sm">Paragraph</Text>
```

## Button with Polymorphism

```tsx
type ButtonBaseProps = {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
  children: ReactNode;
};

type ButtonProps<E extends ElementType = 'button'> = 
  ButtonBaseProps & 
  Omit<ComponentPropsWithoutRef<E>, keyof ButtonBaseProps> & {
    as?: E;
  };

function Button<E extends ElementType = 'button'>({
  as,
  variant = 'primary',
  size = 'md',
  isLoading,
  children,
  ...props
}: ButtonProps<E>) {
  const Component = as || 'button';
  
  return (
    <Component
      className={`btn btn-${variant} btn-${size}`}
      disabled={isLoading}
      {...props}
    >
      {isLoading ? 'Loading...' : children}
    </Component>
  );
}

// As button (default)
<Button onClick={() => {}}>Click</Button>

// As link
<Button as="a" href="/about">About</Button>

// As Next.js Link
import Link from 'next/link';
<Button as={Link} href="/about">About</Button>
```

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*