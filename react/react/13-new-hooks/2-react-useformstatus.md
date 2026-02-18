---
source_course: "react"
source_lesson: "react-useformstatus"
---

# useFormStatus Hook

Access the status of a parent `<form>` from any child component without prop drilling.

## Signature

```jsx
const { pending, data, method, action } = useFormStatus();
```

## Key Properties

- **pending**: `true` while form is submitting
- **data**: FormData being submitted
- **method**: HTTP method ('get' or 'post')
- **action**: Reference to the action function

## Important Rule

`useFormStatus` must be called from a component **inside** a `<form>`. It reads status from the nearest parent form.

## Common Pattern

Create a reusable submit button that knows when its form is submitting:

## Code Examples

**useFormStatus for smart submit buttons**

```tsx
import { useFormStatus } from 'react-dom';

// Reusable submit button - no props needed!
function SubmitButton({ children = 'Submit' }) {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? (
        <>
          <Spinner size="sm" />
          Processing...
        </>
      ) : (
        children
      )}
    </button>
  );
}

// Usage in multiple forms
function ContactForm() {
  return (
    <form action={submitContact}>
      <input name="email" type="email" required />
      <textarea name="message" required />
      <SubmitButton>Send Message</SubmitButton>
    </form>
  );
}

function NewsletterForm() {
  return (
    <form action={subscribeNewsletter}>
      <input name="email" placeholder="Enter email" />
      <SubmitButton>Subscribe</SubmitButton>
    </form>
  );
}

// Advanced: Show what's being submitted
function FormDebug() {
  const { pending, data } = useFormStatus();
  
  if (!pending || !data) return null;
  
  return (
    <pre className="debug">
      Submitting: {JSON.stringify(Object.fromEntries(data))}
    </pre>
  );
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*