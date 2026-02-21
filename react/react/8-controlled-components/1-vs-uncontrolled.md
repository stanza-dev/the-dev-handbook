---
source_course: "react"
source_lesson: "react-controlled-vs-uncontrolled"
---

# Controlled vs Uncontrolled Components

## Introduction

Every form input in a React application faces a fundamental question: who owns the data? In a controlled component, React state is the single source of truth — every keystroke updates state, and the input always reflects that state. In an uncontrolled component, the DOM itself holds the data, and you read it only when needed (typically on form submission). Understanding when to use each pattern is essential for building forms that are both user-friendly and maintainable.

## Key Concepts

- **Controlled component**: An input whose value is driven by React state via the `value` prop and updated through an `onChange` handler.
- **Uncontrolled component**: An input that manages its own state internally via the DOM. You access its value using a ref or FormData.
- **`value` vs `defaultValue`**: Setting `value` makes the input controlled; setting `defaultValue` makes it uncontrolled with an initial value.
- **Single source of truth**: In controlled components, React state is the single source of truth; in uncontrolled, the DOM is.

## Real World Context

A search input with autocomplete suggestions is a perfect use case for controlled components — you need access to the current value on every keystroke to filter suggestions. A simple contact form where you only need the values on submit is a good fit for uncontrolled components, especially with React 19's form actions that receive FormData directly. Most production applications use a mix of both.

## Deep Dive

**Controlled Component:**

```tsx
import { useState } from "react";

function ControlledForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const isValid = email.includes("@") && password.length >= 8;

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log("Submitting:", { email, password });
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
      <button type="submit" disabled={!isValid}>
        Submit
      </button>
      <p>{password.length}/8 characters minimum</p>
    </form>
  );
}
```

**Uncontrolled Component:**

```tsx
import { useRef } from "react";

function UncontrolledForm() {
  const emailRef = useRef<HTMLInputElement>(null);
  const passwordRef = useRef<HTMLInputElement>(null);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log("Submitting:", {
      email: emailRef.current?.value,
      password: passwordRef.current?.value,
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" ref={emailRef} defaultValue="" placeholder="Email" />
      <input type="password" ref={passwordRef} placeholder="Password" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**When to use each:**

| Use Controlled When | Use Uncontrolled When |
|--------------------|-----------------------|
| Real-time validation | Simple forms with minimal validation |
| Conditional field enabling | File inputs (`<input type="file">` is always uncontrolled) |
| Formatting input as user types | Integration with non-React libraries |
| Dynamic form fields | Performance-critical forms with many fields |
| Autocomplete/search suggestions | React 19 form actions with FormData |

## Common Pitfalls

1. **Setting `value` without `onChange`** — If you set the `value` prop but forget the `onChange` handler, the input becomes read-only because React enforces that the input value matches state, but there is no way to update that state.
2. **Switching between controlled and uncontrolled** — Starting with `defaultValue` (uncontrolled) and later setting `value` (controlled), or vice versa, causes a React warning. Decide the pattern upfront and stick with it.

## Best Practices

1. **Default to uncontrolled with React 19 form actions** — For forms that submit data to the server, React 19's `action` prop with FormData eliminates the need for controlled inputs in many cases, reducing boilerplate.
2. **Use controlled only when you need the value during typing** — If you need real-time validation, live search, or input formatting, controlled is the right choice. Otherwise, uncontrolled is simpler.

## Summary

- Controlled components use `value` + `onChange` with React state as the single source of truth; uncontrolled components use `defaultValue` + refs with the DOM as the source.
- Controlled components enable real-time validation and formatting; uncontrolled components are simpler for basic forms.
- React 19 form actions with FormData make uncontrolled patterns more attractive for server-submitted forms.

## Code Examples

**Controlled search input vs uncontrolled form with action**

```tsx
import { useState, useRef } from "react";

// Controlled: React state drives the input
function ControlledSearch() {
  const [query, setQuery] = useState("");
  const results = items.filter((i) => i.includes(query));

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ul>{results.map((r) => <li key={r}>{r}</li>)}</ul>
    </div>
  );
}

// Uncontrolled: DOM holds the value, read on submit
function UncontrolledContact() {
  return (
    <form action={async (formData) => {
      await submitContact(Object.fromEntries(formData));
    }}>
      <input name="email" type="email" defaultValue="" />
      <textarea name="message" defaultValue="" />
      <button type="submit">Send</button>
    </form>
  );
}
```


## Resources

- [Reacting to Input with State](https://react.dev/learn/reacting-to-input-with-state) — Official guide on managing form input state

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*