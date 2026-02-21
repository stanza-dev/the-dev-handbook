---
source_course: "react-intermediate"
source_lesson: "react-usetransition-deep-dive"
---

# useTransition Hook Deep Dive

## Introduction

The `useTransition` hook is your primary tool for keeping the UI responsive during expensive state updates. It returns a pending state and a function to wrap your updates.

## Key Concepts

**Signature:**
```tsx
const [isPending, startTransition] = useTransition();
```

- `isPending`: Boolean indicating if a transition is in progress
- `startTransition`: Function to wrap state updates you want to mark as transitions

## Real World Context

In production apps, useTransition shines for:
- Tab switching with expensive content
- Filtering/sorting large datasets
- Real-time search with instant previews
- Page transitions in single-page apps

## Deep Dive

### How startTransition Works

When you call `startTransition(callback)`, React:

1. Executes the callback synchronously
2. Marks all state updates inside as transitions
3. Starts rendering the new state in the background
4. Can interrupt this render if a more urgent update occurs
5. Shows the old UI until the new one is ready

### The isPending State

`isPending` becomes `true` immediately when you call `startTransition` and stays true until the transition completes. Use it to show loading indicators:

```tsx
<div style={{ opacity: isPending ? 0.5 : 1 }}>
  {/* Content that's being updated */}
</div>
```

### Transitions vs Urgent Updates

React distinguishes between:

| Update Type | Priority | Example | API |
|------------|----------|---------|-----|
| Urgent | High | Typing, clicking | Regular `setState` |
| Transition | Low | Search results, tab content | `startTransition` |

### Important Constraints

1. In React 19, `startTransition` supports async callbacks, allowing you to `await` inside them and have the transition tracked
2. State updates outside `startTransition` in the same event handler are still urgent

```tsx
function handleClick() {
  // This is URGENT - happens immediately
  setInputValue(newValue);
  
  startTransition(() => {
    // This is a TRANSITION - can be interrupted
    setSearchResults(computeResults(newValue));
  });
}
```

## Common Pitfalls

1. **Wrapping the wrong updates**: Only wrap the expensive, deferrable updates - not the urgent ones like input values.
2. **Not providing visual feedback**: Users need to know something is happening. Always use `isPending` to show loading states.
3. **Using with controlled inputs incorrectly**: Never wrap the input's value update in a transition - only wrap derived state.

## Best Practices

- Show visual feedback using `isPending` (opacity, spinners, skeletons)
- Keep input/interaction updates outside transitions
- Use for tab switching, filters, search results, and navigation
- Combine with React.memo to prevent unnecessary child re-renders
- Consider `useDeferredValue` for simpler cases where you don't need `isPending`

## Summary

`useTransition` marks state updates as non-urgent, allowing React to keep the UI responsive. It returns `isPending` for loading states and `startTransition` to wrap your deferrable updates.

## Code Examples

**Tab switching with useTransition for expensive content**

```tsx
import { useState, useTransition, memo } from 'react';

// Memoized expensive component
const SlowList = memo(function SlowList({ text }: { text: string }) {
  const items = [];
  for (let i = 0; i < 500; i++) {
    items.push(<SlowItem key={i} text={text} />);
  }
  return <ul className="slow-list">{items}</ul>;
});

function SlowItem({ text }: { text: string }) {
  // Artificial slowdown
  const startTime = performance.now();
  while (performance.now() - startTime < 1) {}
  return <li>{text}</li>;
}

export function TabContainer() {
  const [tab, setTab] = useState('about');
  const [isPending, startTransition] = useTransition();

  function selectTab(nextTab: string) {
    startTransition(() => {
      setTab(nextTab);
    });
  }

  return (
    <div>
      <nav className="tabs">
        <TabButton
          isActive={tab === 'about'}
          onClick={() => selectTab('about')}
        >
          About
        </TabButton>
        <TabButton
          isActive={tab === 'posts'}
          onClick={() => selectTab('posts')}
        >
          Posts (slow)
        </TabButton>
        <TabButton
          isActive={tab === 'contact'}
          onClick={() => selectTab('contact')}
        >
          Contact
        </TabButton>
      </nav>

      <div className="tab-content" style={{ opacity: isPending ? 0.6 : 1 }}>
        {isPending && <div className="loading-overlay">Loading...</div>}
        {tab === 'about' && <AboutTab />}
        {tab === 'posts' && <SlowList text="Posts" />}
        {tab === 'contact' && <ContactTab />}
      </div>
    </div>
  );
}
```


## Resources

- [Marking a state update as a non-blocking transition](https://react.dev/reference/react/useTransition#marking-a-state-update-as-a-non-blocking-transition) — Official guide on using useTransition

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*