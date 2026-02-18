---
source_course: "react-advanced-apis"
source_lesson: "react-advanced-apis-flush-sync"
---

# flushSync: Forcing Immediate DOM Updates

`flushSync` forces React to flush any pending updates synchronously. This ensures the DOM is updated immediately after the callback.

## When You Need It

React batches state updates for performance. Sometimes you need the DOM to update immediately:

```tsx
import { flushSync } from 'react-dom';

function ScrollToBottom() {
  const [messages, setMessages] = useState([]);
  const listRef = useRef(null);

  const addMessage = (text) => {
    flushSync(() => {
      setMessages([...messages, text]);
    });
    // DOM is updated NOW - we can scroll
    listRef.current.scrollTop = listRef.current.scrollHeight;
  };

  return (
    <div ref={listRef}>
      {messages.map((msg, i) => <p key={i}>{msg}</p>)}
    </div>
  );
}
```

## Without flushSync

```tsx
const addMessage = (text) => {
  setMessages([...messages, text]);
  // DOM NOT updated yet - scroll happens too early!
  listRef.current.scrollTop = listRef.current.scrollHeight;
};
```

## Common Use Cases

### 1. Third-Party Integration

When integrating with non-React code that needs immediate DOM access:

```tsx
function Editor() {
  const [content, setContent] = useState('');
  const editorRef = useRef(null);

  const updateContent = (newContent) => {
    flushSync(() => {
      setContent(newContent);
    });
    // Third-party library needs the DOM updated
    thirdPartyEditor.refresh(editorRef.current);
  };
}
```

### 2. Focus Management

```tsx
function Form() {
  const [showInput, setShowInput] = useState(false);
  const inputRef = useRef(null);

  const handleClick = () => {
    flushSync(() => {
      setShowInput(true);
    });
    // Input exists now, we can focus it
    inputRef.current?.focus();
  };

  return (
    <>
      <button onClick={handleClick}>Add Input</button>
      {showInput && <input ref={inputRef} />}
    </>
  );
}
```

## Performance Warning

⚠️ **Use sparingly!** `flushSync` can significantly hurt performance:

- Breaks React's batching optimization
- Forces synchronous re-renders
- Can cause layout thrashing

```tsx
// 🔴 Bad - unnecessary flushSync
flushSync(() => {
  setCount(count + 1);
});
console.log('Count updated'); // Don't need DOM for this

// ✅ Good - only when DOM access is needed
flushSync(() => {
  setItems([...items, newItem]);
});
scrollToBottom(); // Actually needs the DOM
```

## Suspense Interaction

`flushSync` does not flush Suspense boundaries. They still show fallbacks as needed.

## Resources

- [flushSync API Reference](https://react.dev/reference/react-dom/flushSync) — Official React documentation for flushSync

---

> 📘 *This lesson is part of the [React Advanced APIs](https://stanza.dev/courses/react-advanced-apis) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*