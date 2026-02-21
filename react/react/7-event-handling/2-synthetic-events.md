---
source_course: "react"
source_lesson: "react-synthetic-events"
---

# React Synthetic Events

## Introduction

When you handle an event in React, you are not working directly with the browser's native event object. Instead, React wraps every event in a SyntheticEvent — a cross-browser wrapper that normalizes event properties and behavior across different browsers. This abstraction layer is one of the reasons React applications behave consistently whether your users are on Chrome, Firefox, Safari, or Edge. Understanding SyntheticEvents helps you work with events more effectively and debug event-related issues faster.

## Key Concepts

- **SyntheticEvent**: React's cross-browser wrapper around the native DOM event. It has the same interface as native events (`stopPropagation`, `preventDefault`, `target`, etc.) but works identically across browsers.
- **Event pooling**: In older React versions (before 17), SyntheticEvents were pooled and reused. In React 17+ this is no longer the case, so you can access event properties asynchronously.
- **`nativeEvent`**: Every SyntheticEvent has a `nativeEvent` property that gives you access to the underlying browser event when needed.
- **Event delegation**: React attaches a single event listener to the root of your app (since React 17) and delegates all events through it, rather than attaching listeners to individual DOM nodes.

## Real World Context

Imagine building a drag-and-drop interface that needs precise mouse coordinates. The `clientX` and `clientY` properties might behave differently in older browsers. React's SyntheticEvent normalizes these values so your drag-and-drop code works the same everywhere. When you need browser-specific behavior (like accessing `dataTransfer` for file drops), you can reach through to the `nativeEvent`.

## Deep Dive

SyntheticEvents mirror the native DOM event interface:

```tsx
function EventInspector() {
  const handleClick = (e: React.MouseEvent<HTMLDivElement>) => {
    // Standard event properties (normalized across browsers)
    console.log("Type:", e.type);                    // "click"
    console.log("Target:", e.target);                 // the clicked element
    console.log("CurrentTarget:", e.currentTarget);   // the div with the handler
    console.log("Coordinates:", e.clientX, e.clientY);

    // Access the native browser event when needed
    console.log("Native:", e.nativeEvent);
    console.log("Native type:", e.nativeEvent.type);

    // Standard methods
    e.preventDefault();
    e.stopPropagation();
  };

  return <div onClick={handleClick}>Click to inspect</div>;
}
```

Since React 17+, you can safely access events in async code:

```tsx
function AsyncHandler() {
  const handleChange = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    // In React 17+, this is safe — events are no longer pooled
    await validateAsync(value);
    console.log("Validated:", value);
  };

  return <input onChange={handleChange} />;
}
```

React's event delegation means performance scales well:

```tsx
// React does NOT attach 1000 click handlers to 1000 list items.
// It attaches ONE handler to the root and delegates.
function LargeList({ items }: { items: string[] }) {
  return (
    <ul onClick={(e) => {
      const target = e.target as HTMLElement;
      if (target.tagName === "LI") {
        console.log("Clicked:", target.textContent);
      }
    }}>
      {items.map((item, i) => (
        <li key={i}>{item}</li>
      ))}
    </ul>
  );
}
```

## Common Pitfalls

1. **Assuming events are pooled** — If you learned React before version 17, you might remember needing `e.persist()`. This is no longer necessary. Events are not pooled in modern React and can be accessed asynchronously.
2. **Confusing `e.target` and `e.currentTarget`** — `e.target` is the element that triggered the event (what was clicked), while `e.currentTarget` is the element the handler is attached to. In event delegation patterns, these are different.

## Best Practices

1. **Use `e.currentTarget` for the handler's element** — When your handler is on a parent and events bubble up from children, `e.currentTarget` reliably refers to the element with the handler, while `e.target` could be any child.
2. **Access `nativeEvent` only when necessary** — Stick to the SyntheticEvent API for cross-browser compatibility. Only use `e.nativeEvent` when you need browser-specific properties not available on the synthetic wrapper.

## Summary

- SyntheticEvents are React's cross-browser wrappers around native DOM events, providing a consistent API.
- Event pooling was removed in React 17, so you can safely use event properties in async callbacks.
- React uses event delegation (one listener at the root) for performance, making `e.target` vs `e.currentTarget` an important distinction.

## Code Examples

**Exploring SyntheticEvent properties and behavior**

```tsx
function SyntheticEventDemo() {
  const handleClick = (e: React.MouseEvent) => {
    // SyntheticEvent properties
    console.log("type:", e.type);
    console.log("target:", e.target);
    console.log("currentTarget:", e.currentTarget);

    // Access native event if needed
    console.log("native:", e.nativeEvent);

    // Safe in async (React 17+, no pooling)
    setTimeout(() => {
      console.log("Still accessible:", e.type);
    }, 1000);
  };

  return <button onClick={handleClick}>Inspect Event</button>;
}
```


## Resources

- [React Event Object](https://react.dev/reference/react-dom/components/common#react-event-object) — Official SyntheticEvent reference

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*