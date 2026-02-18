---
source_course: "react-forms"
source_lesson: "react-forms-what-are-uncontrolled-components"
---

# What Are Uncontrolled Components?

An **uncontrolled component** is a form element that maintains its own internal state. Instead of writing an event handler for every state update, you use a **ref** to get form values from the DOM when you need them.

## When to Use Uncontrolled Components

- Simple forms where you only need the value on submit
- Integrating with non-React code
- File inputs (which are always uncontrolled)
- When you want the DOM to handle the form state

## Basic Example

```jsx
import { useRef } from 'react';

function SimpleForm() {
  const inputRef = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    // Get value directly from DOM
    alert('Name: ' + inputRef.current.value);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        ref={inputRef}
        defaultValue="Guest"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

## defaultValue vs value

- `value` makes the input controlled (React controls it)
- `defaultValue` sets an initial value but lets the DOM manage it

```jsx
// Controlled - React manages state
<input value={name} onChange={handleChange} />

// Uncontrolled - DOM manages state
<input defaultValue="initial" ref={inputRef} />
```

## Comparison

| Feature | Controlled | Uncontrolled |
|---------|------------|---------------|
| Value source | React state | DOM |
| Update method | onChange + setState | ref.current.value |
| Instant validation | ✅ Easy | ❌ Complex |
| Conditional disable | ✅ Easy | ❌ Complex |
| Dynamic input | ✅ Easy | ❌ Complex |
| Less code | ❌ | ✅ |

📚 **Learn more**: [Manipulating the DOM with Refs](https://react.dev/learn/manipulating-the-dom-with-refs)

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*