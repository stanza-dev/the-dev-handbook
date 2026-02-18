---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-useformstatus"
---

# useFormStatus: Reading Parent Form State

`useFormStatus` gives you status information about the parent form. It's perfect for building reusable submit buttons and form components.

## Basic Usage

```tsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending, data, method, action } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  );
}

function ContactForm() {
  async function handleSubmit(formData: FormData) {
    await sendMessage(formData);
  }

  return (
    <form action={handleSubmit}>
      <input name="email" type="email" required />
      <textarea name="message" required />
      <SubmitButton /> {/* Automatically knows form state! */}
    </form>
  );
}
```

## The Status Object

```tsx
const status = useFormStatus();

// status.pending: boolean - Is the form submitting?
// status.data: FormData | null - The submitted form data
// status.method: string - 'get' or 'post'
// status.action: function - The form's action function
```

## Important: Must Be Inside the Form

`useFormStatus` reads the status of a **parent** `<form>`. It won't work if called in the same component that renders the form:

```tsx
// 🔴 Wrong - useFormStatus is in the same component as <form>
function Form() {
  const { pending } = useFormStatus(); // Always returns pending: false!
  return (
    <form action={action}>
      <button disabled={pending}>Submit</button>
    </form>
  );
}

// ✅ Correct - useFormStatus is in a child component
function SubmitButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>Submit</button>;
}

function Form() {
  return (
    <form action={action}>
      <SubmitButton />
    </form>
  );
}
```

## Building Reusable Form Components

```tsx
function FormField({ name, label, type = 'text' }: FieldProps) {
  const { pending } = useFormStatus();

  return (
    <div className="field">
      <label htmlFor={name}>{label}</label>
      <input
        id={name}
        name={name}
        type={type}
        disabled={pending}
      />
    </div>
  );
}

function FormActions({ submitText = 'Submit' }) {
  const { pending } = useFormStatus();

  return (
    <div className="actions">
      <button type="reset" disabled={pending}>Reset</button>
      <button type="submit" disabled={pending}>
        {pending ? <Spinner /> : submitText}
      </button>
    </div>
  );
}
```

## Resources

- [useFormStatus API Reference](https://react.dev/reference/react-dom/hooks/useFormStatus) — Official React documentation for useFormStatus hook

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*