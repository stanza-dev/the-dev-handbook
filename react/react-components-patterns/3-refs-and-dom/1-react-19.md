---
source_course: "react-components-patterns"
source_lesson: "react-components-patterns-refs-react-19"
---

# Refs in React 19: Simplified API

React 19 simplifies ref handling - `ref` is now a regular prop. No more `forwardRef` wrapper needed!

## Before React 19: forwardRef

```tsx
// Had to wrap in forwardRef
const Input = forwardRef<HTMLInputElement, InputProps>(
  function Input({ label, ...props }, ref) {
    return (
      <label>
        {label}
        <input ref={ref} {...props} />
      </label>
    );
  }
);
```

## React 19: ref as a Prop

```tsx
type InputProps = {
  label: string;
  ref?: React.Ref<HTMLInputElement>;
} & React.InputHTMLAttributes<HTMLInputElement>;

function Input({ label, ref, ...props }: InputProps) {
  return (
    <label>
      {label}
      <input ref={ref} {...props} />
    </label>
  );
}

// Usage - same as before
function Form() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const focusInput = () => {
    inputRef.current?.focus();
  };
  
  return (
    <>
      <Input label="Name" ref={inputRef} />
      <button onClick={focusInput}>Focus</button>
    </>
  );
}
```

## Ref Cleanup Functions

React 19 also adds cleanup functions for refs:

```tsx
function VideoPlayer({ src }) {
  return (
    <video
      src={src}
      ref={(node) => {
        if (node) {
          // Setup: node is attached
          node.play();
        }
        
        // Return cleanup function
        return () => {
          // Cleanup: node is being removed
          node?.pause();
        };
      }}
    />
  );
}
```

## Multiple Refs

When you need multiple refs on the same element:

```tsx
function useMergedRefs<T>(...refs: React.Ref<T>[]) {
  return useCallback((node: T | null) => {
    refs.forEach(ref => {
      if (typeof ref === 'function') {
        ref(node);
      } else if (ref) {
        (ref as React.MutableRefObject<T | null>).current = node;
      }
    });
  }, refs);
}

// Usage
function Input({ ref, ...props }: InputProps) {
  const internalRef = useRef<HTMLInputElement>(null);
  const mergedRef = useMergedRefs(ref, internalRef);
  
  useEffect(() => {
    // Can use internalRef for internal logic
    console.log(internalRef.current?.value);
  });
  
  return <input ref={mergedRef} {...props} />;
}
```

## Common Use Cases

### Focus Management

```tsx
function SearchForm() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  useEffect(() => {
    // Auto-focus on mount
    inputRef.current?.focus();
  }, []);
  
  return <input ref={inputRef} type="search" />;
}
```

### Scroll Into View

```tsx
function MessageList({ messages }) {
  const endRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    endRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages]);
  
  return (
    <div className="messages">
      {messages.map(msg => <Message key={msg.id} {...msg} />)}
      <div ref={endRef} />
    </div>
  );
}
```

### Measuring Elements

```tsx
function Tooltip({ children, content }) {
  const [rect, setRect] = useState<DOMRect | null>(null);
  const targetRef = useRef<HTMLSpanElement>(null);
  
  useLayoutEffect(() => {
    if (targetRef.current) {
      setRect(targetRef.current.getBoundingClientRect());
    }
  }, []);
  
  return (
    <>
      <span ref={targetRef}>{children}</span>
      {rect && <TooltipContent position={rect}>{content}</TooltipContent>}
    </>
  );
}
```

## Resources

- [Manipulating the DOM with Refs](https://react.dev/learn/manipulating-the-dom-with-refs) — Official React guide on using refs for DOM access

---

> 📘 *This lesson is part of the [React Components & Patterns](https://stanza.dev/courses/react-components-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*