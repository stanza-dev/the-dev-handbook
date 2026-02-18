---
source_course: "react-performance"
source_lesson: "react-performance-virtualization-basics"
---

# Virtualization Basics

Render only visible items for large lists.

## The Problem

```jsx
// ❌ Rendering 10,000 items
function BadList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
// 10,000 DOM nodes = slow initial render, high memory
```

## The Solution: Windowing

Only render items currently in viewport:

```
┌────────────────────┐
│ Hidden items above │  ← Not rendered
├────────────────────┤
│ ████████████████████│  ← Visible
│ ████████████████████│  ← items
│ ████████████████████│  ← rendered
├────────────────────┤
│ Hidden items below │  ← Not rendered
└────────────────────┘
```

## React Window

```bash
npm install react-window
```

```jsx
import { FixedSizeList } from 'react-window';

function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].name}
    </div>
  );
  
  return (
    <FixedSizeList
      height={400}           // Container height
      width="100%"           // Container width
      itemCount={items.length}
      itemSize={35}          // Row height
    >
      {Row}
    </FixedSizeList>
  );
}
```

## Variable Size Items

```jsx
import { VariableSizeList } from 'react-window';

function VariableList({ items }) {
  // Function to get item height
  const getItemSize = (index) => {
    return items[index].isExpanded ? 100 : 35;
  };
  
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].content}
    </div>
  );
  
  return (
    <VariableSizeList
      height={400}
      width="100%"
      itemCount={items.length}
      itemSize={getItemSize}
    >
      {Row}
    </VariableSizeList>
  );
}
```

## Grid Virtualization

```jsx
import { FixedSizeGrid } from 'react-window';

function VirtualizedGrid({ items, columnCount }) {
  const Cell = ({ columnIndex, rowIndex, style }) => {
    const index = rowIndex * columnCount + columnIndex;
    if (index >= items.length) return null;
    
    return (
      <div style={style}>
        {items[index].name}
      </div>
    );
  };
  
  const rowCount = Math.ceil(items.length / columnCount);
  
  return (
    <FixedSizeGrid
      height={400}
      width={800}
      columnCount={columnCount}
      columnWidth={200}
      rowCount={rowCount}
      rowHeight={100}
    >
      {Cell}
    </FixedSizeGrid>
  );
}
```

## With Auto-Sizer

```jsx
import { FixedSizeList } from 'react-window';
import AutoSizer from 'react-virtualized-auto-sizer';

function ResponsiveList({ items }) {
  return (
    <div style={{ height: '100vh' }}>
      <AutoSizer>
        {({ height, width }) => (
          <FixedSizeList
            height={height}
            width={width}
            itemCount={items.length}
            itemSize={35}
          >
            {({ index, style }) => (
              <div style={style}>{items[index].name}</div>
            )}
          </FixedSizeList>
        )}
      </AutoSizer>
    </div>
  );
}
```

## When to Virtualize

| Items | Virtualization |
|-------|----------------|
| < 100 | Not needed |
| 100-500 | Consider it |
| > 500 | Recommended |
| > 1000 | Required |

---

> 📘 *This lesson is part of the [React Performance Deep Dive](https://stanza.dev/courses/react-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*