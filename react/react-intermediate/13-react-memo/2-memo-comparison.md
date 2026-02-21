---
source_course: "react-intermediate"
source_lesson: "react-memo-comparison"
---

# Custom Comparison and Shallow Equality

## Introduction

React.memo's default shallow comparison works well for most cases, but sometimes you need more control. Understanding exactly how shallow comparison works and when to provide a custom comparator is key to using memo effectively.

## Key Concepts

- **Shallow equality**: Compares each prop with `Object.is()`. For primitives, this is a value comparison. For objects and functions, it checks reference identity.
- **Custom comparator**: A function passed as the second argument to memo that receives the previous and next props and returns true to skip re-render.
- **Stable references**: Props that maintain the same reference across renders, achieved with useState initializers, useMemo, and useCallback.

## Real World Context

A chart component receives a `data` prop that is a large array and an `options` prop that is an object. The data rarely changes, but a parent state update causes both to be recreated as new references. You could stabilize them with useMemo, or provide a custom comparator that checks `data.length` and `options.theme` instead of full reference equality.

## Deep Dive

Here is how the default shallow comparison works under the hood:

```tsx
function shallowEqual(prevProps, nextProps) {
  const prevKeys = Object.keys(prevProps);
  const nextKeys = Object.keys(nextProps);
  if (prevKeys.length !== nextKeys.length) return false;
  return prevKeys.every(key => Object.is(prevProps[key], nextProps[key]));
}
```

For a custom comparator:

```tsx
const Chart = memo(
  function Chart({ data, options }: ChartProps) {
    return <canvas>{/* render chart */}</canvas>;
  },
  (prevProps, nextProps) => {
    return (
      prevProps.data.length === nextProps.data.length &&
      prevProps.data === nextProps.data &&
      prevProps.options.theme === nextProps.options.theme
    );
  }
);
```

Note that returning `true` from the comparator means "props are equal, skip re-render" and `false` means "props differ, re-render." This is the opposite of `shouldComponentUpdate` in class components.

The most common approach is not to use custom comparators but rather to fix the root cause by stabilizing props with useMemo and useCallback. Custom comparators can be fragile if you forget to check a new prop that is added later.

## Common Pitfalls

1. **Custom comparator with the wrong return semantics** — Returning true means "equal" (skip render), not "should update." This is the opposite of `shouldComponentUpdate`.
2. **Comparing only some props** — If you add a new prop to the component and forget to update the comparator, the component may not re-render when it should.

## Best Practices

1. **Prefer stabilizing props over custom comparators** — Use useMemo and useCallback to make props referentially stable. This is safer than a comparator that may become stale.
2. **Use custom comparators sparingly** — Only when stabilizing props is impractical, such as when the data comes from outside React.

## Summary

- Shallow comparison checks reference equality for objects and value equality for primitives.
- Custom comparators give fine-grained control but are fragile when props change.
- Prefer stabilizing props with useMemo/useCallback over custom comparison functions.

## Code Examples

**React.memo with a custom comparison function**

```tsx
import { memo } from 'react';

// Custom comparator for memo
const Chart = memo(
  function Chart({ data, theme }: { data: number[]; theme: string }) {
    // Expensive chart rendering
    return <canvas />;
  },
  (prev, next) => {
    // Return true to SKIP re-render
    return (
      prev.data === next.data &&
      prev.theme === next.theme
    );
  }
);
```


## Resources

- [memo](https://react.dev/reference/react/memo) — Official React.memo documentation with custom comparator

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*