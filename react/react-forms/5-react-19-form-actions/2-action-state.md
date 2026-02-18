---
source_course: "react-forms"
source_lesson: "react-forms-use-action-state"
---

# useActionState Hook

`useActionState` is a Hook that manages form submission state, error handling, and the action function.

## Syntax

```jsx
const [state, formAction, isPending] = useActionState(
  actionFunction,
  initialState
);
```

- **state**: Current state (result of last action)
- **formAction**: Action to pass to form's `action` prop
- **isPending**: Boolean indicating if action is running

## Basic Example

```jsx
import { useActionState } from 'react';

function LoginForm() {
  const [error, submitAction, isPending] = useActionState(
    async (previousState, formData) => {
      const email = formData.get('email');
      const password = formData.get('password');
      
      try {
        await login(email, password);
        return null; // No error
      } catch (err) {
        return err.message; // Return error message
      }
    },
    null // Initial state (no error)
  );

  return (
    <form action={submitAction}>
      <input name="email" type="email" required />
      <input name="password" type="password" required />
      
      <button type="submit" disabled={isPending}>
        {isPending ? 'Logging in...' : 'Log In'}
      </button>
      
      {error && <p className="error">{error}</p>}
    </form>
  );
}
```

## With Structured State

```jsx
function AddToCartForm({ productId }) {
  const [state, formAction, isPending] = useActionState(
    async (prevState, formData) => {
      try {
        const result = await addToCart(productId);
        return {
          success: true,
          message: 'Added to cart!',
          cartCount: result.cartCount
        };
      } catch (err) {
        return {
          success: false,
          message: err.message
        };
      }
    },
    { success: null, message: '', cartCount: 0 }
  );

  return (
    <form action={formAction}>
      <button disabled={isPending}>
        {isPending ? 'Adding...' : 'Add to Cart'}
      </button>
      
      {state.success === true && (
        <p className="success">
          {state.message} ({state.cartCount} items)
        </p>
      )}
      
      {state.success === false && (
        <p className="error">{state.message}</p>
      )}
    </form>
  );
}
```

## Action Function Signature

The action function receives:
1. **previousState**: The previous state value
2. **formData**: FormData object from the form

```jsx
async function action(previousState, formData) {
  // previousState is useful for:
  // - Tracking attempt count
  // - Keeping previous values
  // - Building on previous results
  
  return newState; // This becomes the next state
}
```

📚 **Learn more**: [useActionState Reference](https://react.dev/reference/react/useActionState)

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*