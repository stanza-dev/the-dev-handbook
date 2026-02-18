---
source_course: "react-typescript"
source_lesson: "react-typescript-typing-components"
---

# Typing Function Components

There are several ways to type React function components.

## Typing Props Directly

The recommended approach:

```tsx
type GreetingProps = {
  name: string;
  age?: number; // Optional prop
};

function Greeting({ name, age }: GreetingProps) {
  return (
    <div>
      Hello, {name}!
      {age && <span> You are {age} years old.</span>}
    </div>
  );
}
```

## Using React.FC (Avoid)

```tsx
// ❌ Not recommended - implicit children, issues with generics
const Greeting: React.FC<GreetingProps> = ({ name }) => {
  return <div>Hello, {name}!</div>;
};

// ✅ Better - explicit and clear
function Greeting({ name }: GreetingProps) {
  return <div>Hello, {name}!</div>;
}
```

## Default Props

```tsx
type ButtonProps = {
  size?: 'small' | 'medium' | 'large';
  variant?: 'primary' | 'secondary';
  children: React.ReactNode;
};

function Button({
  size = 'medium',
  variant = 'primary',
  children
}: ButtonProps) {
  return (
    <button className={`btn-${size} btn-${variant}`}>
      {children}
    </button>
  );
}
```

## Required vs Optional Props

```tsx
type UserProps = {
  id: string;           // Required
  name: string;         // Required
  email?: string;       // Optional (can be undefined)
  bio?: string | null;  // Optional (can be null)
};
```

## Typing Return Value (Usually Inferred)

```tsx
// Return type is inferred - usually don't need to specify
function Greeting({ name }: { name: string }) {
  return <div>Hello, {name}!</div>;
}

// Explicit return type (rarely needed)
function Greeting({ name }: { name: string }): React.ReactElement {
  return <div>Hello, {name}!</div>;
}

// For components that might return null
function MaybeGreeting({ name }: { name?: string }): React.ReactElement | null {
  if (!name) return null;
  return <div>Hello, {name}!</div>;
}
```

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*