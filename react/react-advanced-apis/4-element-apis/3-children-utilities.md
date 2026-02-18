---
source_course: "react-advanced-apis"
source_lesson: "react-advanced-apis-children-utilities"
---

# Children: Working with props.children

`Children` provides utilities for working with the opaque `props.children` data structure.

## Why Children Utilities?

`props.children` can be:
- A single element
- An array of elements
- A string
- A number
- null/undefined
- A fragment

The `Children` utilities handle all these cases consistently.

## Children.map

Transform each child, handling all edge cases:

```tsx
import { Children } from 'react';

function RowList({ children }) {
  return (
    <div className="rows">
      {Children.map(children, (child, index) => (
        <div className="row" key={index}>
          {child}
        </div>
      ))}
    </div>
  );
}
```

## Children.forEach

Iterate without returning:

```tsx
function validateChildren({ children }) {
  Children.forEach(children, (child) => {
    if (child.type !== ListItem) {
      console.warn('List only accepts ListItem children');
    }
  });
}
```

## Children.count

Count the number of children:

```tsx
function TabList({ children }) {
  const count = Children.count(children);
  
  return (
    <div>
      <span>{count} tabs</span>
      {children}
    </div>
  );
}
```

## Children.only

Assert and return the only child:

```tsx
function OnlyChild({ children }) {
  // Throws if children is not exactly one element
  const child = Children.only(children);
  return <div className="wrapper">{child}</div>;
}
```

## Children.toArray

Convert children to a flat array:

```tsx
function Reverse({ children }) {
  const childArray = Children.toArray(children);
  return <>{childArray.reverse()}</>;
}

function Interleave({ children, separator }) {
  const childArray = Children.toArray(children);
  
  return (
    <>
      {childArray.map((child, i) => (
        <Fragment key={i}>
          {i > 0 && separator}
          {child}
        </Fragment>
      ))}
    </>
  );
}

// Usage
<Interleave separator=" | ">
  <span>A</span>
  <span>B</span>
  <span>C</span>
</Interleave>
// Renders: A | B | C
```

## ⚠️ Limitations and Alternatives

Children utilities have limitations:

1. **Can't see inside fragments** - Fragments are treated as single children
2. **Brittle** - Depends on specific children structure
3. **Hard to type** - TypeScript struggles with children manipulation

### Prefer These Patterns:

```tsx
// 🔴 Manipulating children
function Tabs({ children }) {
  return Children.map(children, (child, i) =>
    cloneElement(child, { index: i })
  );
}

// ✅ Render prop
function Tabs({ items, renderTab }) {
  return items.map((item, i) => renderTab(item, i));
}

// ✅ Compound components with context
function Tabs({ children }) {
  const [activeIndex, setActiveIndex] = useState(0);
  return (
    <TabsContext.Provider value={{ activeIndex, setActiveIndex }}>
      {children}
    </TabsContext.Provider>
  );
}
```

## Resources

- [Children API Reference](https://react.dev/reference/react/Children) — Official React documentation for Children utilities

---

> 📘 *This lesson is part of the [React Advanced APIs](https://stanza.dev/courses/react-advanced-apis) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*