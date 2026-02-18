---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-concurrent-features"
---

# useTransition: Non-Blocking State Updates

React's concurrent features let you mark state updates as non-urgent, keeping your UI responsive during expensive operations.

## The Problem

```tsx
function SearchResults() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    setQuery(value); // Urgent: update input immediately
    setResults(filterResults(value)); // Can be slow with large datasets
  };

  return (
    <>
      <input value={query} onChange={handleChange} />
      <ResultsList results={results} />
    </>
  );
}
```

If `filterResults` is slow, typing feels laggy because both updates block the UI.

## The Solution: useTransition

```tsx
function SearchResults() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    setQuery(value); // Urgent: update immediately

    startTransition(() => {
      // Non-urgent: can be interrupted
      setResults(filterResults(value));
    });
  };

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <Spinner />}
      <ResultsList results={results} />
    </>
  );
}
```

## How It Works

1. **Urgent updates** (outside `startTransition`) render immediately
2. **Transition updates** (inside `startTransition`) can be interrupted
3. If a new urgent update comes in, React abandons the transition and starts fresh
4. `isPending` indicates if a transition is in progress

## Real-World Example: Tab Switching

```tsx
function Tabs() {
  const [tab, setTab] = useState('home');
  const [isPending, startTransition] = useTransition();

  const selectTab = (nextTab: string) => {
    startTransition(() => {
      setTab(nextTab);
    });
  };

  return (
    <>
      <TabButtons
        selectedTab={tab}
        onSelect={selectTab}
        isPending={isPending}
      />
      <Suspense fallback={<Spinner />}>
        {tab === 'home' && <Home />}
        {tab === 'posts' && <Posts />}
        {tab === 'contact' && <Contact />}
      </Suspense>
    </>
  );
}
```

## Best Practices

- Use for state updates that trigger expensive renders
- Don't wrap everything - only non-urgent updates
- Combine with Suspense for data fetching
- Show pending state to users for better UX

## Resources

- [useTransition API Reference](https://react.dev/reference/react/useTransition) — Official React documentation for useTransition hook

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*