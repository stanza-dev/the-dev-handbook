---
source_course: "react"
source_lesson: "react-form-state-management"
---

# Form State Management

## Introduction

As forms grow beyond a couple of inputs, managing state for each field individually becomes tedious. You end up with a `useState` call for every input, a handler for every field, and validation logic scattered across the component. This lesson explores patterns for managing form state efficiently — from single-object state to reducer patterns — so your forms stay clean and maintainable as they scale.

## Key Concepts

- **Object state pattern**: Using a single `useState` with an object to hold all form fields instead of separate state variables per field.
- **Generic change handler**: A single `onChange` handler that uses the input's `name` attribute to update the correct field in state.
- **`useReducer` for complex forms**: Using a reducer to manage form state when you have complex state transitions (reset, multi-step, conditional fields).
- **FormData pattern**: Using React 19 form actions to avoid managing state entirely for server-submitted forms.

## Real World Context

Consider a user registration form with 10+ fields: name, email, password, address, city, state, zip, phone, birthday, and preferences. Creating 10 separate `useState` calls is repetitive. The object state pattern lets you manage all fields in a single state object with a single change handler, reducing code by 80%.

## Deep Dive

**Object state pattern:**

```tsx
function RegistrationForm() {
  const [form, setForm] = useState({
    name: "",
    email: "",
    password: "",
    city: "",
    zipCode: "",
  });

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setForm((prev) => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log("Form data:", form);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" value={form.name} onChange={handleChange} />
      <input name="email" type="email" value={form.email} onChange={handleChange} />
      <input name="password" type="password" value={form.password} onChange={handleChange} />
      <input name="city" value={form.city} onChange={handleChange} />
      <input name="zipCode" value={form.zipCode} onChange={handleChange} />
      <button type="submit">Register</button>
    </form>
  );
}
```

**Reducer pattern for complex forms:**

```tsx
type FormState = {
  values: Record<string, string>;
  errors: Record<string, string>;
  isSubmitting: boolean;
};

type FormAction =
  | { type: "SET_FIELD"; field: string; value: string }
  | { type: "SET_ERROR"; field: string; error: string }
  | { type: "SUBMIT_START" }
  | { type: "SUBMIT_END" }
  | { type: "RESET" };

function formReducer(state: FormState, action: FormAction): FormState {
  switch (action.type) {
    case "SET_FIELD":
      return {
        ...state,
        values: { ...state.values, [action.field]: action.value },
        errors: { ...state.errors, [action.field]: "" },
      };
    case "SET_ERROR":
      return {
        ...state,
        errors: { ...state.errors, [action.field]: action.error },
      };
    case "SUBMIT_START":
      return { ...state, isSubmitting: true };
    case "SUBMIT_END":
      return { ...state, isSubmitting: false };
    case "RESET":
      return { values: {}, errors: {}, isSubmitting: false };
    default:
      return state;
  }
}
```

**React 19 FormData (no state needed):**

```tsx
function SimpleRegistration() {
  return (
    <form action={async (formData: FormData) => {
      const data = Object.fromEntries(formData);
      await registerUser(data);
    }}>
      <input name="name" required />
      <input name="email" type="email" required />
      <input name="password" type="password" required />
      <button type="submit">Register</button>
    </form>
  );
}
```

## Common Pitfalls

1. **Mutating state directly** — Writing `form.name = "new value"` instead of using the setter with spread. Always create a new object: `setForm(prev => ({ ...prev, name: "new value" }))`.
2. **Forgetting the `name` attribute on inputs** — The generic change handler relies on `e.target.name` to know which field to update. Missing name attributes cause silent bugs where changes are lost.

## Best Practices

1. **Match input `name` to state keys** — Keep the input `name` attributes identical to the state object keys so the generic handler `[e.target.name]: e.target.value` works without mapping.
2. **Use FormData for server submissions** — With React 19, prefer form actions and FormData over controlled state when the form data is only needed at submission time.

## Summary

- The object state pattern uses a single `useState` with an object and a generic `onChange` handler to manage multiple form fields efficiently.
- `useReducer` is ideal for complex forms with conditional fields, multi-step flows, or complex validation logic.
- React 19 form actions with FormData let you skip client state entirely for server-submitted forms.

## Code Examples

**Object state pattern with generic change handler**

```tsx
import { useState } from "react";

function ProfileForm() {
  const [form, setForm] = useState({
    name: "",
    bio: "",
    website: "",
  });

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    const { name, value } = e.target;
    setForm((prev) => ({ ...prev, [name]: value }));
  };

  return (
    <form onSubmit={(e) => { e.preventDefault(); console.log(form); }}>
      <input name="name" value={form.name} onChange={handleChange} />
      <textarea name="bio" value={form.bio} onChange={handleChange} />
      <input name="website" value={form.website} onChange={handleChange} />
      <button type="submit">Save</button>
    </form>
  );
}
```


## Resources

- [Updating Objects in State](https://react.dev/learn/updating-objects-in-state) — Official guide on updating object state immutably

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*