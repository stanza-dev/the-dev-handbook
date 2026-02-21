---
source_course: "react-modern-patterns"
source_lesson: "react-actions-intro"
---

# Understanding Actions in React 19

## Introduction

React 19 introduces Actions, a powerful convention that fundamentally changes how you handle asynchronous operations in your components. Instead of manually juggling loading states, error handling, and optimistic updates across multiple useState calls, Actions let React manage all of that for you automatically. If you have ever wrestled with a form submission that required five separate state variables, Actions are going to feel like a breath of fresh air.

## Key Concepts

- **Action**: Any async function used in a transition. When a function inside `startTransition` is async, React treats it as an Action and automatically tracks its pending state.
- **Form Action**: The `action` prop on a `<form>` element that accepts a function receiving `FormData` directly, replacing the traditional `onSubmit` pattern.
- **Pending State**: React automatically sets `isPending` to true while an Action is running and resets it when the Action completes or errors.
- **Progressive Enhancement**: Form Actions work even before JavaScript loads on the page, making your forms functional during hydration.

## Real World Context

Imagine a settings page where users update their profile name, email, and avatar simultaneously. Before Actions, you would need separate loading indicators, error messages, and success toasts for each field, each managed by its own state variable. With Actions, you wrap your async logic in a transition and React handles pending states, error boundaries, and even optimistic UI updates with minimal code.

## Deep Dive

Before React 19, handling an async form submission looked something like this:

```tsx
// Before React 19 — manual state management
function UpdateNameOld() {
  const [name, setName] = useState("");
  const [error, setError] = useState<string | null>(null);
  const [isPending, setIsPending] = useState(false);

  const handleSubmit = async () => {
    setIsPending(true);
    setError(null);
    try {
      await updateName(name);
    } catch (err) {
      setError(err.message);
    } finally {
      setIsPending(false);
    }
  };

  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        {isPending ? "Saving..." : "Update"}
      </button>
      {error && <p className="error">{error}</p>}
    </div>
  );
}
```

With Actions in React 19, you get automatic pending tracking through `useTransition`:

```tsx
import { useState, useTransition } from "react";

function UpdateNameNew() {
  const [name, setName] = useState("");
  const [error, setError] = useState<string | null>(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      const err = await updateName(name);
      if (err) {
        setError(err);
        return;
      }
      setError(null);
    });
  };

  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        {isPending ? "Saving..." : "Update"}
      </button>
      {error && <p className="error">{error}</p>}
    </div>
  );
}
```

You can also use the `action` prop on a form, which receives `FormData` directly:

```tsx
function ContactForm() {
  async function submitForm(formData: FormData) {
    "use server";
    const email = formData.get("email") as string;
    const message = formData.get("message") as string;
    await saveContact({ email, message });
  }

  return (
    <form action={submitForm}>
      <input name="email" type="email" required />
      <textarea name="message" required />
      <button type="submit">Send</button>
    </form>
  );
}
```

The form action pattern supports progressive enhancement: if JavaScript has not loaded yet, the browser can still submit the form natively and the server function will execute.

## Common Pitfalls

1. **Forgetting that Actions are transitions** — Because Actions run inside transitions, state updates within them are batched and non-urgent. If you need an immediate UI update, use `useOptimistic` alongside the Action.
2. **Creating the async function inside render** — If you create a new async function on every render and pass it to a form `action`, it will cause unnecessary re-renders. Hoist the function outside the component or memoize it.

## Best Practices

1. **Prefer `useActionState` for forms** — For form submissions, `useActionState` provides a cleaner API than manually combining `useTransition` with `useState` because it manages state, the form action, and pending status in one hook.
2. **Always handle errors in your action** — Even though Actions manage pending states, you still need to catch and display errors. Return error objects from your action function rather than throwing.

## Summary

- Actions are async functions used in transitions that get automatic pending state management from React.
- The form `action` prop accepts a function that receives `FormData`, replacing the traditional `onSubmit` plus `preventDefault` pattern.
- Actions support progressive enhancement, working before JavaScript hydrates on the page.

## Code Examples

**React 19 Actions with useTransition and form action prop**

```tsx
import { useState, useTransition } from "react";

// React 19 Action pattern with useTransition
function UpdateProfile() {
  const [name, setName] = useState("");
  const [error, setError] = useState<string | null>(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      const result = await updateName(name);
      if (result.error) {
        setError(result.error);
        return;
      }
      setError(null);
    });
  };

  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        {isPending ? "Saving..." : "Update"}
      </button>
      {error && <p className="error">{error}</p>}
    </div>
  );
}

// Form action pattern
function ContactForm() {
  return (
    <form action={async (formData) => {
      const email = formData.get("email");
      await submitContact(email);
    }}>
      <input name="email" type="email" required />
      <button type="submit">Send</button>
    </form>
  );
}
```


## Resources

- [React 19 Blog Post](https://react.dev/blog/2024/12/05/react-19) — Official React 19 announcement covering Actions
- [useTransition Reference](https://react.dev/reference/react/useTransition) — Official useTransition documentation with Action examples

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*