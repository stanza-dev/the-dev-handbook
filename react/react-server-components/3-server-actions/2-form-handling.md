---
source_course: "react-server-components"
source_lesson: "react-server-components-form-handling"
---

# Advanced Form Handling with Server Actions

Learn to build robust forms with validation, pending states, and optimistic updates.

## Using useActionState

`useActionState` manages form state based on the result of a Server Action:

```tsx
'use client';

import { useActionState } from 'react';
import { createUser } from './actions';

type State = {
  error: string | null;
  success: boolean;
};

export function SignupForm() {
  const [state, formAction, isPending] = useActionState<State, FormData>(
    createUser,
    { error: null, success: false }
  );
  
  return (
    <form action={formAction}>
      <input name="email" type="email" disabled={isPending} />
      <input name="password" type="password" disabled={isPending} />
      
      {state.error && <p className="error">{state.error}</p>}
      {state.success && <p className="success">Account created!</p>}
      
      <button disabled={isPending}>
        {isPending ? 'Creating...' : 'Sign Up'}
      </button>
    </form>
  );
}
```

```tsx
// actions.ts
'use server';

export async function createUser(prevState: State, formData: FormData) {
  const email = formData.get('email') as string;
  const password = formData.get('password') as string;
  
  // Validation
  if (!email.includes('@')) {
    return { error: 'Invalid email', success: false };
  }
  
  if (password.length < 8) {
    return { error: 'Password must be at least 8 characters', success: false };
  }
  
  try {
    await db.users.create({ data: { email, password: hash(password) } });
    return { error: null, success: true };
  } catch (e) {
    return { error: 'Email already exists', success: false };
  }
}
```

## Using useFormStatus

`useFormStatus` reads the status of a parent form:

```tsx
'use client';

import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending, data, method } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  );
}

// Usage - SubmitButton must be INSIDE the form
function ContactForm() {
  return (
    <form action={sendMessage}>
      <input name="message" />
      <SubmitButton /> {/* Automatically knows form state */}
    </form>
  );
}
```

## Optimistic Updates with useOptimistic

Show immediate feedback while the action processes:

```tsx
'use client';

import { useOptimistic } from 'react';
import { addTodo } from './actions';

type Todo = { id: string; text: string; pending?: boolean };

export function TodoList({ todos }: { todos: Todo[] }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo: string) => [
      ...state,
      { id: crypto.randomUUID(), text: newTodo, pending: true }
    ]
  );
  
  async function handleSubmit(formData: FormData) {
    const text = formData.get('todo') as string;
    addOptimisticTodo(text); // Show immediately
    await addTodo(text);      // Actually save
  }
  
  return (
    <>
      <form action={handleSubmit}>
        <input name="todo" />
        <button>Add</button>
      </form>
      <ul>
        {optimisticTodos.map(todo => (
          <li 
            key={todo.id}
            style={{ opacity: todo.pending ? 0.5 : 1 }}
          >
            {todo.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

## Non-Form Usage

Server Actions can be called outside forms:

```tsx
'use client';

import { deletePost } from './actions';
import { useTransition } from 'react';

function DeleteButton({ postId }) {
  const [isPending, startTransition] = useTransition();
  
  const handleDelete = () => {
    startTransition(async () => {
      await deletePost(postId);
    });
  };
  
  return (
    <button onClick={handleDelete} disabled={isPending}>
      {isPending ? 'Deleting...' : 'Delete'}
    </button>
  );
}
```

## Resources

- [useActionState](https://react.dev/reference/react/useActionState) — Official React documentation for useActionState
- [useFormStatus](https://react.dev/reference/react-dom/hooks/useFormStatus) — Official React documentation for useFormStatus

---

> 📘 *This lesson is part of the [React Server Components](https://stanza.dev/courses/react-server-components) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*