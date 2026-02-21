---
source_course: "react"
source_lesson: "react-destructuring-props"
---

# Destructuring Props

## Introduction

Destructuring is the standard way to access individual props inside a React component. While it is a plain JavaScript feature (not React-specific), it is so central to React development that mastering it will make your components cleaner and more readable. This lesson covers basic destructuring, default values, rest patterns, and nested destructuring.

## Key Concepts

- **Object Destructuring**: Extracting named properties from the props object directly in the function parameter list.
- **Default Values**: Providing fallback values for optional props using `= value` syntax.
- **Rest Operator**: Collecting remaining props into a separate object using `...rest`.
- **Renaming**: Aliasing a destructured property with `originalName: newName` syntax.

## Real World Context

In a design system, a `TextInput` component might accept its own props (label, error, helperText) plus all native HTML input attributes. Destructuring lets you pull out the custom props and forward everything else to the underlying `<input>` with the rest operator. This is a pattern you will see in every production component library.

## Deep Dive

### Basic Destructuring

Instead of accessing `props.name` and `props.age`, destructure them in the parameter list:

```tsx
// Without destructuring
function Profile(props: { name: string; age: number }) {
  return <p>{props.name} is {props.age}</p>;
}

// With destructuring (cleaner)
function Profile({ name, age }: { name: string; age: number }) {
  return <p>{name} is {age}</p>;
}
```

### Default Values

```tsx
type ToastProps = {
  message: string;
  duration?: number;
  position?: 'top' | 'bottom';
};

function Toast({ message, duration = 3000, position = 'bottom' }: ToastProps) {
  // duration is 3000 and position is 'bottom' unless overridden
  return (
    <div className={`toast toast--${position}`}>
      {message}
    </div>
  );
}
```

### Rest Operator for Forwarding Props

```tsx
type TextInputProps = {
  label: string;
  error?: string;
} & React.ComponentProps<'input'>;

function TextInput({ label, error, ...inputProps }: TextInputProps) {
  return (
    <div className="field">
      <label>{label}</label>
      <input {...inputProps} className={error ? 'input-error' : ''} />
      {error && <span className="error-text">{error}</span>}
    </div>
  );
}

// All standard input attributes are forwarded
<TextInput
  label="Email"
  type="email"
  required
  placeholder="you@example.com"
  error="Invalid email"
/>
```

### Renaming Destructured Props

Occasionally a prop name conflicts with a local variable or you want a more descriptive name:

```tsx
function UserAvatar({ name: userName, size: avatarSize = 48 }: {
  name: string;
  size?: number;
}) {
  return (
    <img
      src={`/api/avatar/${userName}`}
      width={avatarSize}
      height={avatarSize}
      alt={userName}
    />
  );
}
```

## Common Pitfalls

1. **Destructuring too many levels deep** — `{ user: { address: { city } } }` is fragile. If any intermediate property is undefined, you get a runtime error. Destructure one level at a time and use optional chaining for deeper access.
2. **Forgetting to type optional props** — If a prop has a default value, mark it optional with `?` in the type. Otherwise TypeScript requires callers to always provide it, defeating the purpose of the default.

## Best Practices

1. **Always destructure in the parameter list** — This is the idiomatic React style and makes it immediately clear which props a component uses.
2. **Use the rest operator for wrapper components** — When building input wrappers, button wrappers, or other delegating components, `...rest` ensures you forward all standard HTML attributes without listing them manually.

## Summary

- Destructure props in the function parameter for clean, readable component signatures.
- Provide default values for optional props using `= value` syntax in the destructuring pattern.
- Use the rest operator (`...rest`) to collect and forward remaining props to underlying HTML elements.

## Code Examples

**Button component using destructuring with defaults and rest operator**

```tsx
type ButtonProps = {
  variant?: 'primary' | 'secondary' | 'danger';
  isLoading?: boolean;
} & React.ComponentProps<'button'>;

function Button({
  variant = 'primary',
  isLoading = false,
  children,
  disabled,
  ...rest
}: ButtonProps) {
  return (
    <button
      className={`btn btn--${variant}`}
      disabled={disabled || isLoading}
      {...rest}
    >
      {isLoading ? 'Loading...' : children}
    </button>
  );
}
```


## Resources

- [Passing Props to a Component](https://react.dev/learn/passing-props-to-a-component) — Official guide covering all prop patterns including destructuring

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*