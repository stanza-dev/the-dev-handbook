---
source_course: "react"
source_lesson: "react-controlled-vs-uncontrolled"
---

# Controlled vs Uncontrolled Components

## Controlled Components

Form data is handled by React state. The component "controls" the input.

```jsx
<input value={value} onChange={handleChange} />
```

## Uncontrolled Components

Form data is handled by the DOM itself. Use refs to access values.

```jsx
<input defaultValue="foo" ref={inputRef} />
```

## When to Use Each

| Controlled | Uncontrolled |
|------------|---------------|
| Validation on every keystroke | Simple forms |
| Conditional submit button | File inputs |
| Formatting input | Integration with non-React code |

## Code Examples

**Controlled vs uncontrolled form patterns**

```tsx
import { useState, useRef } from 'react';

// Controlled Component
function ControlledForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log({ email, password });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="Password"
      />
      <button type="submit" disabled={!email || !password}>
        Submit
      </button>
    </form>
  );
}

// Uncontrolled Component
function UncontrolledForm() {
  const emailRef = useRef<HTMLInputElement>(null);
  const passwordRef = useRef<HTMLInputElement>(null);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log({
      email: emailRef.current?.value,
      password: passwordRef.current?.value
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" ref={emailRef} defaultValue="" />
      <input type="password" ref={passwordRef} />
      <button type="submit">Submit</button>
    </form>
  );
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*