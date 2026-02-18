---
source_course: "react"
source_lesson: "react-virtualization"
---

# Virtualizing Long Lists

## Introduction

When rendering thousands of items, even fast renders become slow because of the sheer number of DOM elements. Virtualization renders only visible items.

## Key Concepts

**Virtualization** (or windowing) renders only the items currently visible in the viewport, plus a small buffer. As the user scrolls, items are recycled - old ones removed and new ones added.

## Real World Context

Virtualization is essential for:
- Social media feeds with infinite scroll
- Data tables with thousands of rows
- File explorers with many items
- Chat applications with message history
- Any list where users won't view all items at once

## Deep Dive

### How Virtualization Works

1. Calculate which items are visible based on scroll position
2. Render only those items plus a small overscan buffer
3. Use absolute positioning to place items correctly
4. Update on scroll to recycle items

### Popular Libraries

**TanStack Virtual** (recommended):
- Headless - you control the markup
- Works with any list/grid/table
- Small bundle size
- Active development

**react-window** (simpler):
- Fixed and variable size lists
- Grid support
- Smaller API surface
- Lighter weight

**react-virtuoso**:
- Best for dynamic heights
- Built-in grouping/headers
- Chat-like reverse scrolling

### Using TanStack Virtual

```tsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }) {
  const parentRef = useRef<HTMLDivElement>(null);
  
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50, // Estimated row height
    overscan: 5 // Render 5 extra items outside viewport
  });
  
  return (
    <div
      ref={parentRef}
      style={{ height: '400px', overflow: 'auto' }}
    >
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          position: 'relative'
        }}
      >
        {virtualizer.getVirtualItems().map(virtualRow => (
          <div
            key={virtualRow.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              transform: `translateY(${virtualRow.start}px)`
            }}
          >
            {items[virtualRow.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### When to Virtualize

| List Size | Recommendation |
|-----------|----------------|
| < 100 items | Usually no virtualization needed |
| 100-500 items | Consider if items are complex |
| 500-1000 items | Strongly consider |
| > 1000 items | Almost always virtualize |

## Common Pitfalls

1. **Variable heights without measurement**: Virtualization needs to know item heights. For dynamic heights, use libraries that measure.
2. **Breaking accessibility**: Ensure keyboard navigation still works. Some libraries handle this automatically.
3. **Over-virtualizing**: Simple lists with few items don't need virtualization - it adds complexity.

## Best Practices

- Start with TanStack Virtual for most use cases
- Measure real performance before adding virtualization
- Provide accurate size estimates for better scroll behavior
- Use appropriate overscan (5-10 items) to prevent flashing
- Memoize row components for extra performance
- Handle loading states for infinite scroll

## Summary

Virtualization renders only visible items, enabling performant lists with thousands of items. Use TanStack Virtual or react-window. Only virtualize when you have enough items that performance actually suffers.

## Code Examples

**Virtualized list with TanStack Virtual**

```tsx
import { useRef, memo } from 'react';
import { useVirtualizer } from '@tanstack/react-virtual';

type Item = {
  id: string;
  name: string;
  description: string;
};

// Memoized row for extra performance
const Row = memo(function Row({ item }: { item: Item }) {
  return (
    <div className="row">
      <strong>{item.name}</strong>
      <p>{item.description}</p>
    </div>
  );
});

function VirtualizedList({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null);
  
  const rowVirtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 72, // ~72px per row
    overscan: 5
  });

  return (
    <div className="list-container">
      <div className="list-header">
        Showing {items.length.toLocaleString()} items
      </div>
      
      <div
        ref={parentRef}
        className="list-scroll"
        style={{
          height: '500px',
          overflow: 'auto'
        }}
      >
        <div
          style={{
            height: `${rowVirtualizer.getTotalSize()}px`,
            width: '100%',
            position: 'relative'
          }}
        >
          {rowVirtualizer.getVirtualItems().map((virtualRow) => (
            <div
              key={virtualRow.key}
              className="virtual-row"
              style={{
                position: 'absolute',
                top: 0,
                left: 0,
                width: '100%',
                height: `${virtualRow.size}px`,
                transform: `translateY(${virtualRow.start}px)`
              }}
            >
              <Row item={items[virtualRow.index]} />
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

// Usage with 10,000 items - still performant!
function App() {
  const items = useMemo(() =>
    Array.from({ length: 10000 }, (_, i) => ({
      id: `item-${i}`,
      name: `Item ${i + 1}`,
      description: `Description for item ${i + 1}`
    })),
    []
  );
  
  return <VirtualizedList items={items} />;
}
```


## Resources

- [TanStack Virtual](https://tanstack.com/virtual/latest) — TanStack Virtual documentation
- [react-window](https://react-window.vercel.app/) — react-window library documentation

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*