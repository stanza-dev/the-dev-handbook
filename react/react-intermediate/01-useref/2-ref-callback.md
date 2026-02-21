---
source_course: "react-intermediate"
source_lesson: "react-ref-callback"
---

# Ref Callbacks

## Introduction

While `useRef` covers most DOM access scenarios, there are cases where you need more control over when and how a ref is attached. Ref callbacks give you a function-based approach that runs when React attaches or detaches a DOM node. This pattern is essential for measuring elements, managing dynamic lists of refs, and integrating with third-party libraries.

## Key Concepts

- **Ref Callback**: A function passed to the `ref` attribute instead of a ref object. React calls it with the DOM node when attaching and with `null` when detaching.
- **Dynamic Refs**: Managing refs for a variable number of elements (like list items) where you cannot know the count at compile time.
- **Measurement on Mount**: Using a ref callback to measure an element's dimensions immediately when it appears in the DOM.
- **Cleanup in React 19**: In React 19, ref callbacks can return a cleanup function, similar to useEffect.

## Real World Context

An infinite scroll component needs to observe the last item in a list to trigger loading more data. A ref callback can attach an `IntersectionObserver` to the last item and clean it up when the item is removed. A tooltip system needs to measure the anchor element's position to place the tooltip correctly.

## Deep Dive

### Basic Ref Callback

```tsx
function MeasuredBox() {
  const [height, setHeight] = useState(0);

  const measuredRef = (node: HTMLDivElement | null) => {
    if (node) {
      setHeight(node.getBoundingClientRect().height);
    }
  };

  return (
    <div ref={measuredRef}>
      <p>This box is {height}px tall.</p>
      <p>Try adding more content here.</p>
    </div>
  );
}
```

React calls `measuredRef` with the DOM node after the element mounts. When the element unmounts, React calls it with `null`.

### React 19 Cleanup in Ref Callbacks

React 19 allows returning a cleanup function from ref callbacks:

```tsx
function ObservedElement({ onVisible }: { onVisible: () => void }) {
  const observerRef = (node: HTMLDivElement | null) => {
    if (!node) return;

    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) onVisible();
    });
    observer.observe(node);

    // React 19: cleanup function
    return () => observer.disconnect();
  };

  return <div ref={observerRef}>Watch me!</div>;
}
```

### Managing Refs for Dynamic Lists

When rendering a list of items that each need a ref, use a `Map` to store refs by ID:

```tsx
function ImageGallery({ images }: { images: Array<{ id: string; src: string }> }) {
  const itemsRef = useRef<Map<string, HTMLDivElement>>(new Map());

  const scrollToImage = (id: string) => {
    const node = itemsRef.current.get(id);
    node?.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
  };

  return (
    <div className="gallery">
      {images.map(image => (
        <div
          key={image.id}
          ref={(node) => {
            if (node) {
              itemsRef.current.set(image.id, node);
            } else {
              itemsRef.current.delete(image.id);
            }
          }}
        >
          <img src={image.src} alt="" />
        </div>
      ))}
    </div>
  );
}
```

## Common Pitfalls

1. **Creating a new callback function every render** — If the ref callback is an inline arrow function, React calls it with `null` (detach) and then the node (reattach) on every render. For performance-sensitive code, memoize the callback with `useCallback`.
2. **Forgetting the null check** — Ref callbacks receive `null` when the element is removed. Always check before accessing DOM properties.

## Best Practices

1. **Use ref callbacks for measurement and observation** — When you need to know an element's size, position, or visibility as soon as it mounts, a ref callback is more reliable than `useRef` + `useEffect`.
2. **Use a Map for dynamic ref collections** — When the number of refs is determined at runtime (list items), store them in a `Map` keyed by a stable identifier.

## Summary

- Ref callbacks are functions passed to the `ref` attribute that React calls with the DOM node (on attach) and `null` (on detach).
- React 19 supports returning a cleanup function from ref callbacks for automatic resource teardown.
- Use ref callbacks when you need to measure elements or manage refs for dynamic lists.

## Code Examples

**Ref callback with IntersectionObserver for infinite scroll**

```tsx
function LastItemObserver({ items, onLoadMore }: {
  items: string[];
  onLoadMore: () => void;
}) {
  const lastItemRef = (node: HTMLLIElement | null) => {
    if (!node) return;
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) onLoadMore();
    });
    observer.observe(node);
    return () => observer.disconnect(); // React 19 cleanup
  };

  return (
    <ul>
      {items.map((item, i) => (
        <li key={item} ref={i === items.length - 1 ? lastItemRef : undefined}>
          {item}
        </li>
      ))}
    </ul>
  );
}
```


## Resources

- [Manipulating the DOM with Refs](https://react.dev/learn/manipulating-the-dom-with-refs) — Official guide covering ref callbacks and DOM manipulation

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*