---
source_course: "react"
source_lesson: "react-useactionstate"
---

# useActionState Hook

The `useActionState` hook simplifies form handling by managing state, pending status, and errors in one place.

## Signature

```jsx
const [state, formAction, isPending] = useActionState(actionFn, initialState);
```

## Parameters

- **actionFn**: Async function receiving `(previousState, formData)`
- **initialState**: Initial state value

## Returns

- **state**: Current state (or error)
- **formAction**: Action to pass to form
- **isPending**: Loading indicator

## Key Benefits

- Automatic pending state management
- Previous state access in action
- Works with form's `action` prop
- Supports progressive enhancement

## Code Examples

**Complete form with useActionState**

```tsx
import { useActionState } from 'react';

async function updateProfile(prevState, formData) {
  const name = formData.get('name');
  const email = formData.get('email');
  
  // Validate
  if (!name || name.length < 2) {
    return { error: 'Name must be at least 2 characters' };
  }
  
  // Save to server
  const result = await saveProfile({ name, email });
  
  if (!result.success) {
    return { error: result.message };
  }
  
  return { success: true, message: 'Profile updated!' };
}

function ProfileForm() {
  const [state, formAction, isPending] = useActionState(
    updateProfile,
    { error: null, success: false }
  );

  return (
    <form action={formAction}>
      <input name="name" placeholder="Name" required />
      <input name="email" type="email" placeholder="Email" />
      
      <button type="submit" disabled={isPending}>
        {isPending ? 'Saving...' : 'Save Profile'}
      </button>
      
      {state.error && (
        <p className="error">{state.error}</p>
      )}
      {state.success && (
        <p className="success">{state.message}</p>
      )}
    </form>
  );
}
```


## Resources

- [useActionState Reference](https://react.dev/reference/react/useActionState) — Official useActionState documentation

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*