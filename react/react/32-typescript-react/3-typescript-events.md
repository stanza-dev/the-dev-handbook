---
source_course: "react"
source_lesson: "react-typescript-events"
---

# Typing Events and Handlers

## Introduction

Event handling in React requires proper types for both event objects and handler functions to maintain type safety.

## Key Concepts

React provides **synthetic event types** that mirror DOM events: `ChangeEvent`, `MouseEvent`, `FormEvent`, etc. Each is generic over the element type.

## Real World Context

Typed events:
- Catch typos in event properties at compile time
- Enable autocomplete for event methods and properties
- Prevent passing wrong handler types to elements
- Document expected element types

## Deep Dive

### Common Event Types

```tsx
import { 
  ChangeEvent,
  MouseEvent,
  FormEvent,
  KeyboardEvent,
  FocusEvent,
  DragEvent
} from 'react';

// Input change
function handleChange(e: ChangeEvent<HTMLInputElement>) {
  console.log(e.target.value);
}

// Button click
function handleClick(e: MouseEvent<HTMLButtonElement>) {
  console.log(e.clientX, e.clientY);
}

// Form submit
function handleSubmit(e: FormEvent<HTMLFormElement>) {
  e.preventDefault();
  const formData = new FormData(e.currentTarget);
}
```

### Inline Handler Types

TypeScript infers types for inline handlers:

```tsx
<input
  onChange={(e) => {
    // e is automatically typed as ChangeEvent<HTMLInputElement>
    setValue(e.target.value);
  }}
/>
```

### Handler Function Types

When passing handlers as props:

```tsx
type SearchProps = {
  // Full event handler signature
  onChange: (e: ChangeEvent<HTMLInputElement>) => void;
  
  // Or simplified - just the value
  onSearch: (query: string) => void;
};

function Search({ onChange, onSearch }: SearchProps) {
  return (
    <input
      onChange={(e) => {
        onChange(e);
        onSearch(e.target.value);
      }}
    />
  );
}
```

### ComponentProps for Handler Types

```tsx
import { ComponentProps } from 'react';

// Extract the onChange type from input
type InputChangeHandler = ComponentProps<'input'>['onChange'];

// Use it elsewhere
function useInputValue(initialValue: string) {
  const [value, setValue] = useState(initialValue);
  
  const handleChange: InputChangeHandler = (e) => {
    setValue(e.target.value);
  };
  
  return { value, handleChange };
}
```

### Custom Event-like Props

```tsx
type SliderProps = {
  value: number;
  min: number;
  max: number;
  // Custom handler - not a real event
  onChange: (value: number) => void;
};

function Slider({ value, min, max, onChange }: SliderProps) {
  return (
    <input
      type="range"
      value={value}
      min={min}
      max={max}
      onChange={(e) => onChange(Number(e.target.value))}
    />
  );
}
```

## Common Pitfalls

1. **Wrong element type**: `ChangeEvent<HTMLButtonElement>` for an input - buttons don't have `value`.
2. **Using `any` event**: Loses all type benefits. Import the correct event type.
3. **Confusing target vs currentTarget**: `target` can be any descendant, `currentTarget` is always the element the handler is attached to.

## Best Practices

- Use specific event types: `ChangeEvent<HTMLInputElement>` not just `ChangeEvent<HTMLElement>`
- Prefer simplified handler props when you don't need the full event
- Use `ComponentProps<'element'>` to extract handler types
- Access `e.currentTarget` for the element the handler is on
- Type handlers inline when possible to leverage inference

## Summary

React's synthetic events are generic over element types. Use specific event imports and element generics for full type safety. Prefer simplified handler types when the full event isn't needed.

## Code Examples

**Fully typed form with various event handlers**

```tsx
import {
  ChangeEvent,
  FormEvent,
  KeyboardEvent,
  useState
} from 'react';

type FormData = {
  email: string;
  password: string;
};

function LoginForm({ onSubmit }: { onSubmit: (data: FormData) => Promise<void> }) {
  const [formData, setFormData] = useState<FormData>({
    email: '',
    password: ''
  });
  const [isSubmitting, setIsSubmitting] = useState(false);

  // Typed change handler for inputs
  function handleChange(e: ChangeEvent<HTMLInputElement>) {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  }

  // Typed form submission
  async function handleSubmit(e: FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setIsSubmitting(true);
    
    try {
      await onSubmit(formData);
    } finally {
      setIsSubmitting(false);
    }
  }

  // Keyboard event for special handling
  function handleKeyDown(e: KeyboardEvent<HTMLInputElement>) {
    if (e.key === 'Escape') {
      (e.target as HTMLInputElement).blur();
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="email"
        type="email"
        value={formData.email}
        onChange={handleChange}
        onKeyDown={handleKeyDown}
        required
      />
      <input
        name="password"
        type="password"
        value={formData.password}
        onChange={handleChange}
        required
      />
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Signing in...' : 'Sign In'}
      </button>
    </form>
  );
}
```


## Resources

- [DOM Events in TypeScript](https://react.dev/learn/typescript#typing-dom-events) — Official guide to typing DOM events

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*