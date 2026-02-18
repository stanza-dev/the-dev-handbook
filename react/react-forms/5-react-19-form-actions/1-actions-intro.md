---
source_course: "react-forms"
source_lesson: "react-forms-form-actions-intro"
---

# Introduction to Form Actions

React 19 introduces **Form Actions** - a new way to handle form submissions that simplifies state management and works seamlessly with Server Components.

## The Old Way

```jsx
function OldForm() {
  const [isPending, setIsPending] = useState(false);
  const [error, setError] = useState(null);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsPending(true);
    setError(null);
    
    const formData = new FormData(e.target);
    
    try {
      await submitForm(formData);
    } catch (err) {
      setError(err.message);
    } finally {
      setIsPending(false);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      {/* ... */}
    </form>
  );
}
```

## The New Way with Form Actions

```jsx
function NewForm() {
  async function submitAction(formData) {
    // This function receives FormData directly
    await submitForm(formData);
  }

  return (
    <form action={submitAction}>
      <input name="email" type="email" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

## Key Benefits

1. **No `e.preventDefault()`** - React handles it
2. **Automatic FormData** - Passed directly to action
3. **Progressive Enhancement** - Forms work without JS
4. **Pending States** - Built-in with `useFormStatus`
5. **Server Actions** - Works with React Server Components

## How It Works

1. Pass an async function to the `action` prop
2. When form submits, React calls your function with `FormData`
3. React automatically handles pending states
4. Use `useFormStatus` in child components to access pending state

📚 **Learn more**: [React 19 Blog Post](https://react.dev/blog/2024/12/05/react-19)

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*