---
source_course: "react-forms"
source_lesson: "react-forms-use-form-status"
---

# useFormStatus Hook

`useFormStatus` gives you status information about the parent form's submission. It's perfect for creating reusable submit buttons!

## Important Rule

`useFormStatus` must be called from a component **inside** the `<form>`. It doesn't work in the same component that renders the form.

```jsx
// ❌ Won't work - same component as form
function Form() {
  const { pending } = useFormStatus();
  return <form>...</form>;
}

// ✅ Works - child component
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

## What useFormStatus Returns

```jsx
const { pending, data, method, action } = useFormStatus();
```

- **pending**: Boolean, true during submission
- **data**: FormData being submitted (or null)
- **method**: HTTP method ('get' or 'post')
- **action**: Reference to action function

## Reusable Submit Button

```jsx
import { useFormStatus } from 'react-dom';

function SubmitButton({ children, loadingText = 'Submitting...' }) {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? loadingText : children}
    </button>
  );
}

// Usage
function ContactForm() {
  return (
    <form action={submitContact}>
      <input name="message" />
      <SubmitButton loadingText="Sending...">
        Send Message
      </SubmitButton>
    </form>
  );
}
```

## Loading Indicator

```jsx
function FormSpinner() {
  const { pending } = useFormStatus();
  
  if (!pending) return null;
  
  return (
    <div className="spinner" aria-label="Submitting form...">
      ⏳
    </div>
  );
}
```

## Disable Fields During Submission

```jsx
function FormFields() {
  const { pending } = useFormStatus();
  
  return (
    <>
      <input 
        name="email" 
        disabled={pending}
        aria-disabled={pending}
      />
      <textarea 
        name="message" 
        disabled={pending}
        aria-disabled={pending}
      />
    </>
  );
}
```

📚 **Learn more**: [useFormStatus Reference](https://react.dev/reference/react-dom/hooks/useFormStatus)

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*