---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-r19-useactionstate"
---

# useActionState: Form Actions Made Easy

`useActionState` is a Hook that manages form state based on the result of a form action. It's designed to work seamlessly with Server Actions and progressive enhancement.

## Basic Usage

```tsx
import { useActionState } from 'react';

async function submitForm(previousState: State, formData: FormData) {
  const email = formData.get('email') as string;
  
  // Validate
  if (!email.includes('@')) {
    return { error: 'Invalid email', email };
  }
  
  // Submit to server
  await saveEmail(email);
  return { success: true, email };
}

function NewsletterForm() {
  const [state, formAction, isPending] = useActionState(submitForm, {
    error: null,
    success: false,
    email: ''
  });

  return (
    <form action={formAction}>
      <input
        name="email"
        type="email"
        defaultValue={state.email}
        disabled={isPending}
      />
      {state.error && <p className="error">{state.error}</p>}
      {state.success && <p className="success">Subscribed!</p>}
      <button type="submit" disabled={isPending}>
        {isPending ? 'Subscribing...' : 'Subscribe'}
      </button>
    </form>
  );
}
```

## How It Works

1. **Initial render**: `state` equals `initialState`
2. **Form submission**: Action is called with `(previousState, formData)`
3. **Action completes**: `state` updates to the returned value
4. **isPending**: `true` while the action is running

## With Server Actions

```tsx
// actions.ts
'use server';

export async function createPost(prevState: State, formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  try {
    await db.posts.create({ title, content });
    return { success: true, error: null };
  } catch (e) {
    return { success: false, error: 'Failed to create post' };
  }
}

// Component
'use client';
import { useActionState } from 'react';
import { createPost } from './actions';

function PostForm() {
  const [state, action, pending] = useActionState(createPost, {
    success: false,
    error: null
  });

  return (
    <form action={action}>
      <input name="title" required />
      <textarea name="content" required />
      <button disabled={pending}>
        {pending ? 'Creating...' : 'Create Post'}
      </button>
      {state.error && <p>{state.error}</p>}
    </form>
  );
}
```

## Progressive Enhancement

Forms using `useActionState` work even before JavaScript loads:

```tsx
const [state, action] = useActionState(serverAction, initialState, '/posts');
//                                                                  ^^^^^^^^
//                                                     Optional permalink for no-JS
```

## Resources

- [useActionState API Reference](https://react.dev/reference/react/useActionState) — Official React documentation for useActionState hook

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*