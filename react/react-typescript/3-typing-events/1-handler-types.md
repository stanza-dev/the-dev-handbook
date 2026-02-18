---
source_course: "react-typescript"
source_lesson: "react-typescript-event-handler-types"
---

# Event Handler Types

React provides types for all DOM events.

## Inline Event Handlers

Types are inferred when writing inline:

```tsx
function Button() {
  return (
    <button
      onClick={(e) => {
        // e is React.MouseEvent<HTMLButtonElement>
        console.log(e.currentTarget.innerText);
      }}
    >
      Click me
    </button>
  );
}
```

## Extracted Handler Functions

Need explicit types when extracting:

```tsx
function Form() {
  // Type the event parameter
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const form = e.currentTarget;
    // ...
  };

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input onChange={handleChange} />
    </form>
  );
}
```

## Common Event Types

```tsx
// Mouse events
React.MouseEvent<HTMLButtonElement>
React.MouseEvent<HTMLDivElement>

// Form events
React.FormEvent<HTMLFormElement>
React.ChangeEvent<HTMLInputElement>
React.ChangeEvent<HTMLTextAreaElement>
React.ChangeEvent<HTMLSelectElement>

// Keyboard events
React.KeyboardEvent<HTMLInputElement>

// Focus events
React.FocusEvent<HTMLInputElement>

// Drag events
React.DragEvent<HTMLDivElement>

// Clipboard events
React.ClipboardEvent<HTMLInputElement>
```

## Event Handler Type Shortcuts

```tsx
// Instead of typing the event...
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {};

// ...you can type the handler function itself:
const handleClick: React.MouseEventHandler<HTMLButtonElement> = (e) => {
  // e is properly typed!
};

// Common handler types
React.MouseEventHandler<HTMLButtonElement>
React.ChangeEventHandler<HTMLInputElement>
React.FormEventHandler<HTMLFormElement>
React.KeyboardEventHandler<HTMLInputElement>
```

## Generic Input Handler

```tsx
function Form() {
  const [form, setForm] = useState({ name: '', email: '' });

  // Works for any input with matching name attribute
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setForm(prev => ({ ...prev, [name]: value }));
  };

  return (
    <form>
      <input name="name" value={form.name} onChange={handleChange} />
      <input name="email" value={form.email} onChange={handleChange} />
    </form>
  );
}
```

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*