---
source_course: "react"
source_lesson: "react-conditional-rendering"
---

# Conditional Rendering in JSX

React lets you conditionally render components using JavaScript operators.

## Methods

1. **If statements** (outside JSX)
2. **Ternary operator**: `condition ? <A /> : <B />`
3. **Logical AND**: `condition && <Component />`
4. **Nullish rendering**: Return `null` to render nothing

## Code Examples

**Different conditional rendering patterns**

```tsx
function Notification({ count }) {
  return (
    <div>
      {/* Ternary for either/or */}
      {count > 0 ? (
        <span className="badge">{count}</span>
      ) : (
        <span>No notifications</span>
      )}
      
      {/* Logical AND for show/hide */}
      {count > 99 && <span>99+</span>}
    </div>
  );
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*