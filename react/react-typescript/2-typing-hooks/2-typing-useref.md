---
source_course: "react-typescript"
source_lesson: "react-typescript-typing-useref"
---

# Typing useRef

useRef has different typing depending on its use case.

## DOM Element Refs

For accessing DOM elements, use `null` as initial value:

```tsx
import { useRef, useEffect } from 'react';

function TextInput() {
  // HTMLInputElement | null
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    // Need to check for null
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} />;
}
```

## Common DOM Types

```tsx
// Input elements
const inputRef = useRef<HTMLInputElement>(null);
const textareaRef = useRef<HTMLTextAreaElement>(null);
const selectRef = useRef<HTMLSelectElement>(null);

// Container elements
const divRef = useRef<HTMLDivElement>(null);
const formRef = useRef<HTMLFormElement>(null);

// Media elements
const videoRef = useRef<HTMLVideoElement>(null);
const canvasRef = useRef<HTMLCanvasElement>(null);

// Button
const buttonRef = useRef<HTMLButtonElement>(null);
```

## Mutable Value Refs

For storing mutable values (not DOM elements):

```tsx
// Note: no null - this is a mutable ref
const countRef = useRef<number>(0);
const timerRef = useRef<NodeJS.Timeout | null>(null);
const prevValueRef = useRef<string>('');

function Timer() {
  const timerRef = useRef<NodeJS.Timeout | null>(null);

  const startTimer = () => {
    timerRef.current = setInterval(() => {
      console.log('tick');
    }, 1000);
  };

  const stopTimer = () => {
    if (timerRef.current) {
      clearInterval(timerRef.current);
    }
  };
}
```

## Ref vs State

```tsx
// Ref: Value persists, no re-render on change
const renderCount = useRef(0);
renderCount.current += 1; // No re-render

// State: Causes re-render
const [count, setCount] = useState(0);
setCount(c => c + 1); // Re-renders
```

## Forwarding Refs with Types

```tsx
import { forwardRef } from 'react';

type InputProps = {
  label: string;
  type?: string;
};

const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, type = 'text' }, ref) => {
    return (
      <label>
        {label}
        <input ref={ref} type={type} />
      </label>
    );
  }
);

// Usage
function Form() {
  const inputRef = useRef<HTMLInputElement>(null);
  return <Input ref={inputRef} label="Name" />;
}
```

📚 **Learn more**: [Referencing Values with Refs](https://react.dev/learn/referencing-values-with-refs)

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*