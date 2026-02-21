---
source_course: "react-modern-patterns"
source_lesson: "react-ref-as-prop"
---

# ref as a Regular Prop

## Introduction

For years, passing a ref to a child component required wrapping it in `forwardRef`, adding boilerplate and complexity to otherwise simple components. React 19 finally eliminates this friction: function components can now receive `ref` as a regular prop, just like any other prop. This is one of the most celebrated quality-of-life improvements in the release, simplifying component authoring and reducing the number of React-specific patterns developers need to memorize.

## Key Concepts

- **ref as prop**: In React 19, `ref` is passed as a standard prop to function components. No wrapper function is needed.
- **`forwardRef` deprecation path**: `forwardRef` still works in React 19 for backward compatibility, but it is no longer necessary and will be deprecated in a future version.
- **Cleanup functions**: React 19 also introduces ref cleanup functions. The function you pass to `ref` can return a cleanup function, similar to `useEffect`.
- **TypeScript types**: The `ref` prop type is `React.Ref<T>` where `T` is the element type.

## Real World Context

Consider a design system library with dozens of input components — text inputs, selects, textareas, date pickers — all of which need to expose a DOM ref for focus management, measurement, and third-party library integration. With `forwardRef`, every single component needed an extra wrapper and a different component signature. In React 19, these components are just regular functions with `ref` in their props, making the codebase simpler and more consistent.

## Deep Dive

Before React 19, you had to use `forwardRef`:

```tsx
// React 18 — forwardRef wrapper required
import { forwardRef, useRef } from "react";

const TextInput = forwardRef<HTMLInputElement, { label: string }>(
  function TextInput({ label }, ref) {
    return (
      <label>
        {label}
        <input ref={ref} />
      </label>
    );
  }
);
```

In React 19, ref is just another prop:

```tsx
// React 19 — no wrapper needed
function TextInput({ label, ref }: { label: string; ref?: React.Ref<HTMLInputElement> }) {
  return (
    <label>
      {label}
      <input ref={ref} />
    </label>
  );
}

// Usage is identical
function Form() {
  const inputRef = useRef<HTMLInputElement>(null);

  return (
    <div>
      <TextInput ref={inputRef} label="Username" />
      <button onClick={() => inputRef.current?.focus()}>
        Focus Input
      </button>
    </div>
  );
}
```

React 19 also adds ref cleanup functions:

```tsx
function MeasuredDiv({ ref }: { ref?: React.Ref<HTMLDivElement> }) {
  return (
    <div ref={(node) => {
      // Setup: node is attached
      if (node) {
        console.log("Attached:", node.getBoundingClientRect());
      }
      // Cleanup: return a function
      return () => {
        console.log("Detached");
      };
    }}>
      Measured content
    </div>
  );
}
```

A codemod is available to automatically migrate `forwardRef` usage: `npx @next/codemod@canary react-19/replace-forwardref`.

## Common Pitfalls

1. **Destructuring `ref` and spreading the rest** — When using `{ ref, ...rest }`, ensure you do not accidentally spread `ref` onto a native element via `...rest` because that would create a conflicting ref. Destructure `ref` explicitly.
2. **Mixing old and new patterns** — If you wrap a component in `forwardRef` AND accept `ref` as a prop, the behavior is undefined. Choose one approach per component.

## Best Practices

1. **Stop using forwardRef in new code** — For any new component in React 19, accept `ref` as a prop directly. Only keep `forwardRef` in existing code until you can migrate it.
2. **Use the codemod for migration** — Run the official codemod to automatically convert `forwardRef` wrappers to the new prop-based pattern across your entire codebase.

## Summary

- React 19 lets function components receive `ref` as a regular prop, eliminating the need for `forwardRef`.
- `forwardRef` still works but will be deprecated in a future React version. New code should use the prop-based pattern.
- Ref cleanup functions are a new React 19 feature that lets ref callbacks return a cleanup function, similar to useEffect.

## Code Examples

**ref as a regular prop without forwardRef in React 19**

```tsx
// React 19: ref is just a prop
function TextInput({ label, ref, ...props }: {
  label: string;
  ref?: React.Ref<HTMLInputElement>;
} & React.InputHTMLAttributes<HTMLInputElement>) {
  return (
    <label>
      {label}
      <input ref={ref} {...props} />
    </label>
  );
}

// Usage
function Form() {
  const inputRef = useRef<HTMLInputElement>(null);
  return (
    <div>
      <TextInput ref={inputRef} label="Email" type="email" />
      <button onClick={() => inputRef.current?.focus()}>Focus</button>
    </div>
  );
}
```


## Resources

- [React 19 Upgrade Guide](https://react.dev/blog/2024/04/25/react-19-upgrade-guide) — Migration guide covering ref as prop and other changes

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*