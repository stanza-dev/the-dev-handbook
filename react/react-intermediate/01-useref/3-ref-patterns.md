---
source_course: "react-intermediate"
source_lesson: "react-ref-patterns"
---

# Ref Patterns

## Introduction

Beyond basic DOM access and value storage, refs enable several advanced patterns in React. This lesson explores forwarding refs to child components, combining refs, and using the `useImperativeHandle` hook to expose a custom API from a child component. These patterns are essential for building reusable component libraries and integrating with imperative APIs.

## Key Concepts

- **forwardRef**: A React API that lets a component pass a ref it receives down to a child DOM element.
- **useImperativeHandle**: A hook that customizes what value a ref exposes to the parent, hiding internal implementation details.
- **Combining Refs**: Attaching multiple refs to a single element using a ref callback.
- **Ref Forwarding in Libraries**: How design systems expose controlled access to internal DOM nodes.

## Real World Context

A `TextInput` component in a design system wraps a native `<input>` with a label, error message, and styling. The consuming code sometimes needs to call `.focus()` on the underlying input (for example, focusing the first invalid field after form validation). Ref forwarding lets the consumer pass a ref that reaches through to the internal `<input>` element.

## Deep Dive

### Forwarding Refs

In React 19, function components can accept a `ref` prop directly without wrapping in `forwardRef`:

```tsx
// React 19: ref is a regular prop
function TextInput({ label, ref, ...props }: {
  label: string;
  ref?: React.Ref<HTMLInputElement>;
} & React.ComponentProps<'input'>) {
  return (
    <div className="field">
      <label>{label}</label>
      <input ref={ref} {...props} />
    </div>
  );
}

// Parent can now focus the internal input
function LoginForm() {
  const emailRef = useRef<HTMLInputElement>(null);

  const handleSubmit = () => {
    if (!emailRef.current?.value) {
      emailRef.current?.focus();
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <TextInput label="Email" ref={emailRef} type="email" />
      <button type="submit">Login</button>
    </form>
  );
}
```

### useImperativeHandle

Sometimes you want to expose only specific methods to the parent rather than the entire DOM node:

```tsx
import { useRef, useImperativeHandle } from 'react';

type ModalHandle = {
  open: () => void;
  close: () => void;
};

function Modal({ ref, children }: {
  ref?: React.Ref<ModalHandle>;
  children: React.ReactNode;
}) {
  const [isOpen, setIsOpen] = useState(false);
  const dialogRef = useRef<HTMLDialogElement>(null);

  useImperativeHandle(ref, () => ({
    open: () => {
      setIsOpen(true);
      dialogRef.current?.showModal();
    },
    close: () => {
      setIsOpen(false);
      dialogRef.current?.close();
    },
  }));

  return (
    <dialog ref={dialogRef}>
      {children}
    </dialog>
  );
}

// Parent gets a clean API instead of raw DOM access
function App() {
  const modalRef = useRef<ModalHandle>(null);

  return (
    <>
      <button onClick={() => modalRef.current?.open()}>Open Modal</button>
      <Modal ref={modalRef}>
        <h2>Settings</h2>
        <button onClick={() => modalRef.current?.close()}>Close</button>
      </Modal>
    </>
  );
}
```

### Combining Multiple Refs

When you need to attach both a local ref and a forwarded ref to the same element:

```tsx
function mergeRefs<T>(...refs: Array<React.Ref<T> | undefined>) {
  return (node: T | null) => {
    refs.forEach(ref => {
      if (typeof ref === 'function') ref(node);
      else if (ref) (ref as React.MutableRefObject<T | null>).current = node;
    });
  };
}
```

## Common Pitfalls

1. **Exposing the entire DOM node** — Forwarding a ref to the raw DOM element lets consumers call any DOM method, potentially breaking your component's invariants. Use `useImperativeHandle` to limit the exposed API.
2. **Forgetting that ref is not available on first render** — `ref.current` is `null` until React mounts the element. Always check for null or access refs inside effects and event handlers.

## Best Practices

1. **Use React 19's ref prop** — In React 19, `forwardRef` is no longer necessary. Accept `ref` as a regular prop for cleaner component signatures.
2. **Prefer `useImperativeHandle` for component libraries** — Expose only the methods consumers need (focus, scroll, open, close) rather than the entire DOM node.

## Summary

- React 19 lets components accept `ref` as a regular prop, replacing the need for `forwardRef`.
- `useImperativeHandle` customizes what a ref exposes, keeping component internals encapsulated.
- Use ref callbacks to combine multiple refs on a single element or integrate with observation APIs.

## Code Examples

**useImperativeHandle to expose a custom reset API to the parent**

```tsx
import { useRef, useImperativeHandle, useState } from 'react';

type CounterHandle = { reset: () => void; getValue: () => number };

function Counter({ ref }: { ref?: React.Ref<CounterHandle> }) {
  const [count, setCount] = useState(0);

  useImperativeHandle(ref, () => ({
    reset: () => setCount(0),
    getValue: () => count,
  }));

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  );
}

function App() {
  const counterRef = useRef<CounterHandle>(null);
  return (
    <>
      <Counter ref={counterRef} />
      <button onClick={() => counterRef.current?.reset()}>
        Reset from parent
      </button>
    </>
  );
}
```


## Resources

- [useImperativeHandle Documentation](https://react.dev/reference/react/useImperativeHandle) — Official useImperativeHandle API reference

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*