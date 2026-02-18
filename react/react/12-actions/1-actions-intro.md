---
source_course: "react"
source_lesson: "react-actions-intro"
---

# Actions in React 19

Actions revolutionize how we handle async operations in React. Instead of manually tracking pending states, errors, and optimistic updates, Actions handle these automatically.

## What Changes?

**Before React 19**, you needed:
- `isPending` state for loading indicators
- `error` state for error handling
- Manual state management for each async operation

**With Actions**, React manages:
- Pending states automatically via `useTransition`
- Error handling with built-in patterns
- Form submissions with the `action` prop

## Form Actions

Forms now accept an `action` prop that receives FormData directly:

```jsx
<form action={submitAction}>
```

## Code Examples

**Comparing old vs new async handling patterns**

```tsx
import { useState, useTransition } from 'react';

// Before React 19 - manual state management
function OldWay() {
  const [name, setName] = useState('');
  const [error, setError] = useState(null);
  const [isPending, setIsPending] = useState(false);

  const handleSubmit = async () => {
    setIsPending(true);
    const error = await updateName(name);
    setIsPending(false);
    if (error) setError(error);
  };

  return (/* ... */);
}

// React 19 - with useTransition Actions
function NewWay() {
  const [name, setName] = useState('');
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = () => {
    startTransition(async () => {
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      }
      // Success handling
    });
  };

  return (
    <div>
      <input 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
      />
      <button onClick={handleSubmit} disabled={isPending}>
        {isPending ? 'Saving...' : 'Update'}
      </button>
      {error && <p className="error">{error}</p>}
    </div>
  );
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*