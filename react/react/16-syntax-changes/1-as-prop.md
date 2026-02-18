---
source_course: "react"
source_lesson: "react-ref-as-prop"
---

# No More forwardRef!

In React 19, function components can receive `ref` as a regular prop. No wrapper needed.

## Before (React 18)

```tsx
const Input = forwardRef((props, ref) => (
  <input ref={ref} {...props} />
));
```

## After (React 19)

```tsx
function Input({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}
```

## Migration

- `forwardRef` still works but is no longer needed
- Future versions will deprecate `forwardRef`
- Codemod available for automatic migration

## Code Examples

**ref as a prop without forwardRef**

```tsx
// React 19 - ref is just another prop!
function TextInput({ ref, label, ...props }) {
  return (
    <label>
      {label}
      <input ref={ref} {...props} />
    </label>
  );
}

// Usage - works exactly the same
function Form() {
  const inputRef = useRef(null);
  
  function focusInput() {
    inputRef.current?.focus();
  }
  
  return (
    <div>
      <TextInput 
        ref={inputRef} 
        label="Username" 
        placeholder="Enter username"
      />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
}

// TypeScript types
type InputProps = {
  ref?: React.Ref<HTMLInputElement>;
  label: string;
} & React.InputHTMLAttributes<HTMLInputElement>;
```


## Resources

- [React 19 Upgrade Guide](https://react.dev/blog/2024/04/25/react-19-upgrade-guide) — Migration guide for React 19

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*