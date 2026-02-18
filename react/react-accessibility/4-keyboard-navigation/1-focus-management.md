---
source_course: "react-accessibility"
source_lesson: "react-accessibility-focus-management"
---

# Focus Management

Proper focus management is essential for keyboard users.

## What's Focusable by Default?

- Links (`<a href>`)
- Buttons (`<button>`)
- Form inputs (`<input>`, `<select>`, `<textarea>`)
- Elements with `tabIndex="0"`

## tabIndex Values

```jsx
// tabIndex="0" - Add to tab order (in DOM order)
<div tabIndex={0}>Focusable div</div>

// tabIndex="-1" - Focusable via JavaScript, not Tab key
<div tabIndex={-1}>Programmatically focusable</div>

// ❌ Avoid positive tabIndex - breaks natural order
<button tabIndex={3}>Don't do this</button>
```

## Managing Focus with useRef

```jsx
import { useRef, useEffect } from 'react';

function SearchBox() {
  const inputRef = useRef(null);

  useEffect(() => {
    // Focus input on mount
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} type="search" />;
}
```

## Focus After Actions

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);
  const addButtonRef = useRef(null);

  const deleteTodo = (id) => {
    setTodos(todos.filter(t => t.id !== id));
    // Return focus to a logical place
    addButtonRef.current?.focus();
  };

  return (
    <div>
      <button ref={addButtonRef}>Add Todo</button>
      {todos.map(todo => (
        <TodoItem 
          key={todo.id} 
          onDelete={() => deleteTodo(todo.id)}
        />
      ))}
    </div>
  );
}
```

## Focus on Route Change

```jsx
import { useEffect, useRef } from 'react';
import { useLocation } from 'react-router-dom';

function PageWrapper({ children }) {
  const mainRef = useRef(null);
  const location = useLocation();

  useEffect(() => {
    // Focus main content on route change
    mainRef.current?.focus();
  }, [location.pathname]);

  return (
    <main ref={mainRef} tabIndex={-1}>
      {children}
    </main>
  );
}
```

## Skip Links

Allow keyboard users to skip navigation:

```jsx
function Layout({ children }) {
  return (
    <>
      <a href="#main-content" className="skip-link">
        Skip to main content
      </a>
      <nav>{/* Navigation */}</nav>
      <main id="main-content" tabIndex={-1}>
        {children}
      </main>
    </>
  );
}
```

```css
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  padding: 8px;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*