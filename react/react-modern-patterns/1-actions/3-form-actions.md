---
source_course: "react-modern-patterns"
source_lesson: "react-form-actions"
---

# Form Actions and Progressive Enhancement

## Introduction

React 19 lets you pass an async function directly to the `action` prop on a `<form>` element. This is a major shift from the traditional `onSubmit` handler pattern because it enables progressive enhancement out of the box: your forms can work before JavaScript even loads on the page. This lesson explores how form actions function, how they integrate with the rest of the Actions ecosystem, and why they represent the future of form handling in React.

## Key Concepts

- **Form `action` prop**: Accepts an async function that receives a `FormData` object when the form is submitted.
- **Progressive Enhancement**: The ability for a form to function (submit data to the server) even before client-side JavaScript has loaded or hydrated.
- **FormData API**: The browser-native `FormData` object that collects all named inputs from a form on submission.
- **Action chaining**: Combining form actions with `useActionState`, `useOptimistic`, and `useFormStatus` for a complete submission experience.

## Real World Context

Think about a checkout page on an e-commerce site. Users on slow connections might try to submit their order before React has fully hydrated. With traditional `onSubmit` handlers, that click does nothing because the JavaScript handler is not attached yet. With form actions, the browser falls back to a native form submission, ensuring the order goes through. Once JavaScript loads, subsequent interactions become fully interactive with pending states and optimistic updates.

## Deep Dive

The simplest form action is a function passed to the `action` prop:

```tsx
function SimpleForm() {
  return (
    <form action={async (formData: FormData) => {
      const email = formData.get("email") as string;
      await subscribeToNewsletter(email);
    }}>
      <input name="email" type="email" required />
      <button type="submit">Subscribe</button>
    </form>
  );
}
```

For server-side execution, you mark the function with `"use server"`:

```tsx
// actions.ts
"use server";

export async function subscribe(formData: FormData) {
  const email = formData.get("email") as string;
  await db.subscribers.create({ data: { email } });
}
```

```tsx
import { subscribe } from "./actions";

function NewsletterForm() {
  return (
    <form action={subscribe}>
      <input name="email" type="email" required />
      <button type="submit">Subscribe</button>
    </form>
  );
}
```

You can combine form actions with `useActionState` for state management and `useFormStatus` for pending indicators in child components:

```tsx
import { useActionState } from "react";
import { useFormStatus } from "react-dom";
import { subscribe } from "./actions";

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? "Subscribing..." : "Subscribe"}
    </button>
  );
}

function NewsletterForm() {
  const [state, formAction] = useActionState(subscribe, null);

  return (
    <form action={formAction}>
      <input name="email" type="email" required />
      <SubmitButton />
      {state?.error && <p>{state.error}</p>}
    </form>
  );
}
```

## Common Pitfalls

1. **Forgetting `name` attributes on inputs** — The `FormData` object only includes inputs that have a `name` attribute. Omitting it means `formData.get("field")` returns `null`.
2. **Mixing `onSubmit` and `action`** — While both can coexist, using `onSubmit` with `e.preventDefault()` will block the action from firing. Choose one pattern or the other.

## Best Practices

1. **Use hidden inputs for non-user data** — Pass IDs or tokens via `<input type="hidden" name="postId" value={id} />` so your action function receives all necessary data through FormData.
2. **Validate on both client and server** — Client-side validation improves UX, but server-side validation inside the action function is essential for security since FormData can be manipulated.

## Summary

- The form `action` prop accepts an async function that receives `FormData`, replacing the `onSubmit` pattern.
- Form actions enable progressive enhancement, letting forms work before JavaScript loads.
- Combine form actions with `useActionState` and `useFormStatus` for a complete form experience with state management and pending indicators.

## Code Examples

**Server Function form action with useActionState and useFormStatus**

```tsx
// actions.ts
"use server";

export async function createPost(prevState: any, formData: FormData) {
  const title = formData.get("title") as string;
  const body = formData.get("body") as string;

  if (!title || title.length < 3) {
    return { error: "Title must be at least 3 characters" };
  }

  await db.posts.create({ data: { title, body } });
  revalidatePath("/posts");
  return { error: null, success: true };
}

// CreatePostForm.tsx
"use client";
import { useActionState } from "react";
import { useFormStatus } from "react-dom";
import { createPost } from "./actions";

function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? "Creating..." : "Create Post"}
    </button>
  );
}

export function CreatePostForm() {
  const [state, formAction] = useActionState(createPost, null);

  return (
    <form action={formAction}>
      <input name="title" placeholder="Post title" required />
      <textarea name="body" placeholder="Write your post..." />
      <SubmitButton />
      {state?.error && <p className="error">{state.error}</p>}
    </form>
  );
}
```


## Resources

- [React 19 Form Actions](https://react.dev/reference/react-dom/components/form) — Official React form component documentation with action prop

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*