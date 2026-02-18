---
source_course: "react"
source_lesson: "react-memo-basics"
---

# React.memo

`memo` is a higher-order component that memoizes your component.

## How It Works

```jsx
const MemoizedComponent = memo(MyComponent);
```

The memoized component will only re-render if its props change (shallow comparison).

## When to Use

- Component renders often with same props
- Component is expensive to render
- Parent re-renders frequently but child doesn't need to

## Code Examples

**Preventing re-renders with memo**

```tsx
import { memo, useState } from 'react';

// Expensive component to render
const ExpensiveList = memo(function ExpensiveList({ items }) {
  console.log('ExpensiveList rendered');
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
});

function App() {
  const [count, setCount] = useState(0);
  const [items] = useState([{ id: 1, name: 'Item 1' }]);

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      {/* Won't re-render when count changes! */}
      <ExpensiveList items={items} />
    </div>
  );
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*