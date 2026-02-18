---
source_course: "react"
source_lesson: "react-useeffect-cleanup"
---

# Effect Cleanup

Return a function from your effect to clean up:

```jsx
useEffect(() => {
  // Setup
  return () => {
    // Cleanup
  };
}, [deps]);
```

## When Cleanup Runs

1. Before the effect runs again (when deps change)
2. When the component unmounts

## Common Cleanup Patterns

- Unsubscribe from events
- Cancel network requests
- Clear timers
- Close connections

## Code Examples

**Cleanup with event listeners**

```tsx
function WindowSize() {
  const [size, setSize] = useState({ 
    width: window.innerWidth, 
    height: window.innerHeight 
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };

    window.addEventListener('resize', handleResize);
    
    // Cleanup: remove listener on unmount
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []); // Empty deps = setup once, cleanup on unmount

  return <p>{size.width} x {size.height}</p>;
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*