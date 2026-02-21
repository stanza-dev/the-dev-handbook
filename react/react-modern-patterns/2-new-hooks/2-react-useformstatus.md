---
source_course: "react-modern-patterns"
source_lesson: "react-useformstatus"
---

# useFormStatus for Nested Components

## Introduction

One of the most tedious patterns in React form handling has been passing `isPending` or `isSubmitting` props through multiple layers of components. The `useFormStatus` hook from `react-dom` eliminates this prop drilling entirely. Any component rendered inside a `<form>` can access the parent form's submission status directly, making it trivial to build reusable submit buttons, loading overlays, and input disabling logic.

## Key Concepts

- **`useFormStatus()`**: A hook from `react-dom` that reads the status of the nearest parent `<form>` element.
- **`pending`**: A boolean that is `true` while the form's action is executing.
- **`data`**: The `FormData` object being submitted (or `null` if no submission is in progress).
- **`method`**: The HTTP method of the submission (`"get"` or `"post"`).
- **`action`**: A reference to the function passed to the form's `action` prop.

## Real World Context

Imagine you are building a design system with a universal `<SubmitButton>` component used across dozens of forms in your application. Before `useFormStatus`, each form would need to track its own `isPending` state and pass it down as a prop to the button. With `useFormStatus`, your submit button component reads the pending state from its parent form automatically — zero props required.

## Deep Dive

The hook is imported from `react-dom`, not `react`:

```tsx
import { useFormStatus } from "react-dom";

function SubmitButton({ children = "Submit" }: { children?: React.ReactNode }) {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? "Processing..." : children}
    </button>
  );
}
```

You can use this component in any form without passing any props:

```tsx
function LoginForm() {
  return (
    <form action={loginAction}>
      <input name="email" type="email" required />
      <input name="password" type="password" required />
      <SubmitButton>Log In</SubmitButton>
    </form>
  );
}

function SignupForm() {
  return (
    <form action={signupAction}>
      <input name="email" type="email" required />
      <input name="password" type="password" required />
      <input name="confirmPassword" type="password" required />
      <SubmitButton>Create Account</SubmitButton>
    </form>
  );
}
```

You can also build more advanced components that use the `data` property to show what is being submitted:

```tsx
function SubmissionPreview() {
  const { pending, data } = useFormStatus();
  if (!pending || !data) return null;

  return (
    <div className="preview">
      Submitting: {JSON.stringify(Object.fromEntries(data))}
    </div>
  );
}
```

**Critical rule**: `useFormStatus` must be called from a component that is rendered *inside* a `<form>`. Calling it in the same component that renders the `<form>` element will not work because the hook looks for a parent form in the DOM tree.

## Common Pitfalls

1. **Calling `useFormStatus` in the form component itself** — The hook reads from a *parent* form. If you call it in the same component where the `<form>` is rendered, there is no parent form to read from and `pending` will always be `false`.
2. **Forgetting to import from `react-dom`** — The hook lives in `react-dom`, not `react`. Importing from the wrong package will give you an "is not a function" error.

## Best Practices

1. **Create a reusable SubmitButton component** — Build one component that uses `useFormStatus` and reuse it across your entire application. This centralizes your loading state UI logic.
2. **Disable all form inputs during submission** — Use `useFormStatus` in a wrapper component to disable all inputs while pending, preventing users from editing fields during submission.

## Summary

- `useFormStatus` from `react-dom` reads the submission status of the nearest parent form without any prop drilling.
- It returns `pending`, `data`, `method`, and `action` for the current form submission.
- The hook must be called from a child component inside a `<form>`, not in the component rendering the form itself.

## Code Examples

**Reusable SubmitButton and FormFields using useFormStatus**

```tsx
import { useFormStatus } from "react-dom";

// Reusable submit button — no props needed
function SubmitButton({ children = "Submit" }: { children?: React.ReactNode }) {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? "Processing..." : children}
    </button>
  );
}

// Reusable form field disabler
function FormFields({ children }: { children: React.ReactNode }) {
  const { pending } = useFormStatus();
  return (
    <fieldset disabled={pending} style={{ opacity: pending ? 0.6 : 1 }}>
      {children}
    </fieldset>
  );
}

// Usage
function ContactForm() {
  return (
    <form action={submitContact}>
      <FormFields>
        <input name="email" type="email" required />
        <textarea name="message" required />
      </FormFields>
      <SubmitButton>Send Message</SubmitButton>
    </form>
  );
}
```


## Resources

- [useFormStatus Reference](https://react.dev/reference/react-dom/hooks/useFormStatus) — Official useFormStatus hook documentation

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*