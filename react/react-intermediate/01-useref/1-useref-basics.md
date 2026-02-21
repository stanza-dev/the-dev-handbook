---
source_course: "react-intermediate"
source_lesson: "react-useref-basics"
---

# useRef Basics

## Introduction

While `useState` is the go-to hook for data that affects what appears on screen, sometimes you need to hold onto a value without triggering a re-render. The `useRef` hook returns a mutable object that persists for the lifetime of the component. Its two primary uses are accessing DOM elements directly and storing mutable values that do not need to trigger re-renders.

## Key Concepts

- **Ref Object**: An object with a single `.current` property that you can read and write without causing re-renders.
- **DOM Access**: Attach a ref to a JSX element via the `ref` attribute to get a direct handle to the underlying DOM node.
- **Persistent Value**: Unlike a regular variable declared in the function body, a ref's value survives across re-renders.
- **No Re-render on Change**: Changing `ref.current` does not cause the component to re-render, unlike state.

## Real World Context

A video player component needs to call `.play()` and `.pause()` on the underlying `<video>` DOM element. You cannot do this with JSX alone — you need a direct reference. A stopwatch needs to store an interval ID so it can clear the timer later. Neither of these requires a re-render when the value changes, making `useRef` the right tool.

## Deep Dive

### Accessing DOM Elements

```tsx
import { useRef, useEffect } from 'react';

function AutoFocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} placeholder="I focus automatically" />;
}
```

When React renders the `<input>`, it sets `inputRef.current` to the actual DOM node. You can then call DOM methods like `.focus()`, `.scrollIntoView()`, or read properties like `.getBoundingClientRect()`.

### Storing Mutable Values

```tsx
function Stopwatch() {
  const [time, setTime] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  const intervalRef = useRef<ReturnType<typeof setInterval> | null>(null);

  const start = () => {
    if (isRunning) return;
    setIsRunning(true);
    intervalRef.current = setInterval(() => {
      setTime(prev => prev + 1);
    }, 1000);
  };

  const stop = () => {
    if (!isRunning) return;
    setIsRunning(false);
    if (intervalRef.current) clearInterval(intervalRef.current);
  };

  const reset = () => {
    stop();
    setTime(0);
  };

  return (
    <div>
      <p>{time}s</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

The interval ID is stored in a ref because changing it should not cause a re-render. It is purely a bookkeeping value needed to clear the timer.

### Tracking Previous Values

```tsx
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T | undefined>(undefined);

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current; // Returns the value from the previous render
}

function PriceDisplay({ price }: { price: number }) {
  const prevPrice = usePrevious(price);
  const direction = prevPrice !== undefined && price > prevPrice ? 'up' : 'down';

  return <span className={`price price--${direction}`}>${price}</span>;
}
```

### useState vs useRef Comparison

| Feature | useState | useRef |
|---------|----------|--------|
| Triggers re-render | Yes | No |
| Returns | [value, setter] | { current: value } |
| Best for | Data displayed in UI | DOM nodes, timers, previous values |

## Common Pitfalls

1. **Reading ref.current during render** — Refs are not tracked by React. Reading `ref.current` in the JSX return statement can show stale data because changing it does not trigger a re-render. Use state for values displayed in the UI.
2. **Setting ref.current in the render phase** — Modifying a ref during render (outside of an effect or event handler) can cause issues with concurrent features. Set refs in effects or callbacks.

## Best Practices

1. **Use refs for values the UI does not depend on** — Timer IDs, previous values, external library instances, and DOM nodes are ideal ref candidates.
2. **Prefer state when the value affects the rendered output** — If changing a value should update what the user sees, it must be state, not a ref.

## Summary

- `useRef` returns a `{ current: value }` object that persists across renders without causing re-renders.
- Attach refs to JSX elements to access DOM nodes directly for imperative operations.
- Use refs for mutable values like timer IDs and previous values that do not need to trigger UI updates.

## Code Examples

**Using useRef to control a video element with play/pause**

```tsx
import { useRef, useEffect } from 'react';

function VideoPlayer({ src }: { src: string }) {
  const videoRef = useRef<HTMLVideoElement>(null);

  const handlePlay = () => videoRef.current?.play();
  const handlePause = () => videoRef.current?.pause();

  return (
    <div>
      <video ref={videoRef} src={src} />
      <button onClick={handlePlay}>Play</button>
      <button onClick={handlePause}>Pause</button>
    </div>
  );
}
```


## Resources

- [useRef Documentation](https://react.dev/reference/react/useRef) — Official useRef API reference
- [Manipulating the DOM with Refs](https://react.dev/learn/manipulating-the-dom-with-refs) — Official guide on using refs for DOM access

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*