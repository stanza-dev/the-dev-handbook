---
source_course: "react-forms"
source_lesson: "react-forms-what-are-controlled-components"
---

# What Are Controlled Components?

In HTML, form elements like `<input>`, `<textarea>`, and `<select>` typically maintain their own state and update it based on user input. In React, we often want to have **single source of truth** - meaning React state should be the source of truth for form data.

A **controlled component** is a form element whose value is controlled by React state. The form element's value is set by state, and changes are handled through event handlers that update that state.

## Why Use Controlled Components?

1. **Single Source of Truth**: The React state is always the current value
2. **Validation**: You can validate input on every keystroke
3. **Conditional Disabling**: Easily disable submit buttons based on form validity
4. **Enforced Input Formats**: Format input as the user types (e.g., phone numbers)
5. **Dynamic Inputs**: React to changes immediately

## Basic Example

```jsx
import { useState } from 'react';

function NameForm() {
  const [name, setName] = useState('');

  function handleChange(e) {
    setName(e.target.value);
  }

  return (
    <input
      type="text"
      value={name}        // Controlled by state
      onChange={handleChange}  // Updates state
    />
  );
}
```

The key insight is that `value={name}` makes React the source of truth. The input will always display whatever is in `name` state.

## The Data Flow

1. User types in the input
2. `onChange` event fires with the new value
3. Event handler calls `setName` with `e.target.value`
4. React re-renders the component
5. Input displays the new value from state

This circular flow happens so fast it feels instantaneous, but understanding it is crucial for mastering forms in React.

📚 **Learn more**: [Reacting to Input with State](https://react.dev/learn/reacting-to-input-with-state)

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*