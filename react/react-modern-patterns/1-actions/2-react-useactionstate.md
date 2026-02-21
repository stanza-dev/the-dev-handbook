---
source_course: "react-modern-patterns"
source_lesson: "react-useactionstate"
---

# useActionState Hook

## Introduction

The `useActionState` hook is React 19's all-in-one solution for form state management. It combines the responsibilities of tracking form state, providing a dispatchable action, and surfacing a pending indicator into a single, elegant hook call. If you have ever built a form that needed loading spinners, error messages, and success feedback, this hook eliminates the boilerplate that used to require three or four separate hooks.

## Key Concepts

- **Action Function**: An async function with the signature `(previousState, formData) => newState`. It receives the last state and the submitted FormData, and returns the next state.
- **Initial State**: The starting value before any submission, typically `{ error: null, success: false }` or similar.
- **`formAction`**: A stable reference you pass to the form's `action` prop. React wires it up so that submitting the form invokes your action function.
- **`isPending`**: A boolean that is `true` while the action is executing and `false` otherwise.

## Real World Context

Consider a comment system where users post replies to articles. You need to validate the comment length, send it to the server, show a spinner while saving, display any validation error, and clear the form on success. With `useActionState`, the entire flow is handled by one hook. The action function receives the previous state (so you can accumulate errors or messages) and the FormData, then returns the new state that React applies to the component.

## Deep Dive

Here is the hook signature:

```tsx
const [state, formAction, isPending] = useActionState(actionFn, initialState);
```

The `actionFn` is called with two arguments: the previous state and the FormData from the form submission. It must return the next state (or a Promise that resolves to it).

```tsx
import { useActionState } from "react";

async function updateProfile(
  prevState: { error: string | null; success: boolean },
  formData: FormData
) {
  const name = formData.get("name") as string;
  const email = formData.get("email") as string;

  if (!name || name.length < 2) {
    return { error: "Name must be at least 2 characters", success: false };
  }

  const result = await saveProfile({ name, email });

  if (!result.ok) {
    return { error: result.message, success: false };
  }

  return { error: null, success: true };
}

function ProfileForm() {
  const [state, formAction, isPending] = useActionState(updateProfile, {
    error: null,
    success: false,
  });

  return (
    <form action={formAction}>
      <input name="name" placeholder="Name" required />
      <input name="email" type="email" placeholder="Email" />
      <button type="submit" disabled={isPending}>
        {isPending ? "Saving..." : "Save Profile"}
      </button>
      {state.error && <p className="error">{state.error}</p>}
      {state.success && <p className="success">Profile updated!</p>}
    </form>
  );
}
```

A critical benefit is that the previous state is always available inside the action function. This means you can implement multi-step forms, accumulate validation errors, or track submission counts without any external state.

## Common Pitfalls

1. **Forgetting to return state from the action** — If your action function does not return a value, the state becomes `undefined`, which can cause runtime errors in your JSX.
2. **Confusing parameter order** — The action function receives `(previousState, formData)`, not `(formData, previousState)`. Swapping them is a common mistake that leads to cryptic errors when you try to call `.get()` on the state object.

## Best Practices

1. **Use typed state objects** — Define a TypeScript type for your state (e.g., `{ error: string | null; success: boolean }`) so the compiler catches issues at build time rather than runtime.
2. **Combine with `useFormStatus`** — Place your submit button in a child component that calls `useFormStatus` for a reusable pending indicator pattern that works across all your forms.

## Summary

- `useActionState` returns `[state, formAction, isPending]` and manages the entire form lifecycle.
- The action function signature is `(previousState, formData) => newState`, giving you access to both the prior state and the submitted data.
- It integrates with the form `action` prop for progressive enhancement and works seamlessly with Server Functions.

## Code Examples

**Complete form with useActionState managing state and pending**

```tsx
import { useActionState } from "react";

async function updateProfile(
  prevState: { error: string | null; success: boolean },
  formData: FormData
) {
  const name = formData.get("name") as string;
  if (!name || name.length < 2) {
    return { error: "Name must be at least 2 characters", success: false };
  }
  const result = await saveProfile({ name });
  if (!result.ok) {
    return { error: result.message, success: false };
  }
  return { error: null, success: true };
}

function ProfileForm() {
  const [state, formAction, isPending] = useActionState(updateProfile, {
    error: null,
    success: false,
  });

  return (
    <form action={formAction}>
      <input name="name" placeholder="Name" required />
      <button type="submit" disabled={isPending}>
        {isPending ? "Saving..." : "Save"}
      </button>
      {state.error && <p className="error">{state.error}</p>}
      {state.success && <p className="success">Saved!</p>}
    </form>
  );
}
```


## Resources

- [useActionState Reference](https://react.dev/reference/react/useActionState) — Official useActionState documentation

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*