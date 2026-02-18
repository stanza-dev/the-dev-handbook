---
source_course: "react-performance"
source_lesson: "react-performance-virtualization-advanced"
---

# Advanced Virtualization

Complex virtualization scenarios.

## React Virtuoso

```bash
npm install react-virtuoso
```

```jsx
import { Virtuoso } from 'react-virtuoso';

// Simpler API, more features
function VirtuosoList({ items }) {
  return (
    <Virtuoso
      style={{ height: '400px' }}
      data={items}
      itemContent={(index, item) => (
        <div className="item">{item.name}</div>
      )}
    />
  );
}
```

## Variable Height with Measurement

```jsx
import { Virtuoso } from 'react-virtuoso';

function DynamicHeightList({ items }) {
  return (
    <Virtuoso
      data={items}
      itemContent={(index, item) => (
        <div className="item">
          <h3>{item.title}</h3>
          <p>{item.description}</p> {/* Variable content */}
        </div>
      )}
    />
  );
}
// Virtuoso automatically measures item heights
```

## Grouped Lists

```jsx
import { GroupedVirtuoso } from 'react-virtuoso';

function GroupedList({ groups }) {
  const groupCounts = groups.map(g => g.items.length);
  const allItems = groups.flatMap(g => g.items);
  
  return (
    <GroupedVirtuoso
      style={{ height: '400px' }}
      groupCounts={groupCounts}
      groupContent={(index) => (
        <div className="group-header">
          {groups[index].name}
        </div>
      )}
      itemContent={(index) => (
        <div className="item">
          {allItems[index].name}
        </div>
      )}
    />
  );
}
```

## Infinite Scrolling

```jsx
import { Virtuoso } from 'react-virtuoso';
import { useState, useCallback } from 'react';

function InfiniteList() {
  const [items, setItems] = useState(initialItems);
  const [loading, setLoading] = useState(false);
  
  const loadMore = useCallback(async () => {
    if (loading) return;
    setLoading(true);
    
    const newItems = await fetchMoreItems(items.length);
    setItems(prev => [...prev, ...newItems]);
    setLoading(false);
  }, [items.length, loading]);
  
  return (
    <Virtuoso
      style={{ height: '100vh' }}
      data={items}
      endReached={loadMore}
      itemContent={(index, item) => (
        <div className="item">{item.name}</div>
      )}
      components={{
        Footer: () => (
          loading ? <div>Loading...</div> : null
        ),
      }}
    />
  );
}
```

## Sticky Headers

```jsx
import { Virtuoso } from 'react-virtuoso';

function StickyHeaderList({ items }) {
  return (
    <Virtuoso
      data={items}
      topItemCount={1} // Keep first item sticky
      itemContent={(index, item) => (
        <div className="item">{item.name}</div>
      )}
      components={{
        TopItemList: ({ children, ...props }) => (
          <div {...props} className="sticky-header">
            {children}
          </div>
        ),
      }}
    />
  );
}
```

## Performance Tips

```jsx
// 1. Memoize row components
const Row = memo(function Row({ item }) {
  return <div>{item.name}</div>;
});

// 2. Use stable keys
itemContent={(index, item) => (
  <Row key={item.id} item={item} />
)}

// 3. Avoid inline objects in styles
const rowStyle = { padding: 10 }; // Outside component

// 4. Debounce scroll handlers
const handleRangeChange = useMemo(
  () => debounce((range) => {
    // Track visible range
  }, 100),
  []
);
```

---

> 📘 *This lesson is part of the [React Performance Deep Dive](https://stanza.dev/courses/react-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*