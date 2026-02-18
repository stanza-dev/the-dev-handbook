---
source_course: "react-advanced-apis"
source_lesson: "react-advanced-apis-clone-element"
---

# cloneElement: Creating Modified Copies

`cloneElement` creates a new React element using an existing element as a starting point, allowing you to override props.

## Basic Usage

```tsx
import { cloneElement } from 'react';

const originalElement = <Button color="blue">Click</Button>;

const clonedElement = cloneElement(
  originalElement,
  { color: 'red', size: 'large' } // Override/add props
);

// Result: <Button color="red" size="large">Click</Button>
```

## Common Use Cases

### 1. Adding Props to Children

```tsx
function List({ children, onItemClick }) {
  return (
    <ul>
      {Children.map(children, (child, index) =>
        cloneElement(child, {
          onClick: () => onItemClick(index),
          'data-index': index
        })
      )}
    </ul>
  );
}

// Usage
<List onItemClick={handleClick}>
  <li>Item 1</li>
  <li>Item 2</li>
</List>
```

### 2. Injecting Refs

```tsx
function FocusManager({ children }) {
  const ref = useRef(null);
  
  useEffect(() => {
    ref.current?.focus();
  }, []);
  
  return cloneElement(children, { ref });
}

// Usage
<FocusManager>
  <input placeholder="Auto-focused" />
</FocusManager>
```

### 3. Wrapping with Additional Behavior

```tsx
function Tooltip({ children, content }) {
  const [show, setShow] = useState(false);
  
  return (
    <>
      {cloneElement(children, {
        onMouseEnter: () => setShow(true),
        onMouseLeave: () => setShow(false)
      })}
      {show && <TooltipContent>{content}</TooltipContent>}
    </>
  );
}
```

## Replacing Children

Pass new children as the third argument:

```tsx
const original = <div className="box">Old content</div>;

const cloned = cloneElement(
  original,
  null, // Keep existing props
  'New content', // Replace children
  <span>More content</span>
);
```

## ⚠️ When NOT to Use cloneElement

cloneElement makes data flow harder to trace. Prefer alternatives:

### Instead of cloneElement:

```tsx
// 🔴 Implicit prop injection
function Parent({ children }) {
  return cloneElement(children, { theme: 'dark' });
}

// ✅ Explicit render prop
function Parent({ children }) {
  return children({ theme: 'dark' });
}

// ✅ Context
function Parent({ children }) {
  return (
    <ThemeContext.Provider value="dark">
      {children}
    </ThemeContext.Provider>
  );
}
```

## Resources

- [cloneElement API Reference](https://react.dev/reference/react/cloneElement) — Official React documentation for cloneElement

---

> 📘 *This lesson is part of the [React Advanced APIs](https://stanza.dev/courses/react-advanced-apis) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*