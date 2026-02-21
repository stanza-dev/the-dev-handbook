---
source_course: "react"
source_lesson: "react-children-prop"
---

# The Children Prop

## Introduction

The `children` prop is one of React's most powerful composition tools. It lets a component wrap arbitrary content provided by its parent, making it possible to build flexible layouts, modals, cards, and other container components. This lesson explores how `children` works and the patterns that make it useful.

## Key Concepts

- **children Prop**: A special prop that contains the JSX placed between a component's opening and closing tags.
- **React.ReactNode**: The TypeScript type for `children`, which includes JSX elements, strings, numbers, arrays, fragments, null, and undefined.
- **Wrapper Components**: Components whose primary job is to provide layout, styling, or behavior around their children.
- **Composition**: Using `children` to inject specific content into generic container components.

## Real World Context

Every application has reusable containers: modals, sidebars, tooltips, error boundaries, page layouts. A `Modal` component should not know whether it contains a login form, a confirmation dialog, or a video player. By accepting `children`, the modal handles the overlay, animation, and close behavior while the parent decides what goes inside.

## Deep Dive

### Basic Usage

Anything between a component's opening and closing tags becomes its `children` prop:

```tsx
function Panel({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <section className="panel">
      <h2>{title}</h2>
      <div className="panel-body">{children}</div>
    </section>
  );
}

function App() {
  return (
    <Panel title="Account Settings">
      <p>Update your email and password.</p>
      <button>Save Changes</button>
    </Panel>
  );
}
```

### Modal Example

```tsx
function Modal({ isOpen, onClose, children }: {
  isOpen: boolean;
  onClose: () => void;
  children: React.ReactNode;
}) {
  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal" onClick={e => e.stopPropagation()}>
        <button className="modal-close" onClick={onClose} aria-label="Close">
          ×
        </button>
        {children}
      </div>
    </div>
  );
}

// Usage — the modal does not care what is inside it
<Modal isOpen={showConfirm} onClose={() => setShowConfirm(false)}>
  <h2>Are you sure?</h2>
  <p>This action cannot be undone.</p>
  <button onClick={handleDelete}>Delete</button>
</Modal>
```

### Children as a Render Fence

An important property of `children` is that the parent renders the children JSX, not the wrapper. This means state changes inside the wrapper do not re-render the children unless they consume that state. This makes `children` a natural performance boundary.

```tsx
function ScrollTracker({ children }: { children: React.ReactNode }) {
  const [scrollY, setScrollY] = useState(0);

  useEffect(() => {
    const handler = () => setScrollY(window.scrollY);
    window.addEventListener('scroll', handler);
    return () => window.removeEventListener('scroll', handler);
  }, []);

  return (
    <div>
      <div className="scroll-indicator">Scrolled: {scrollY}px</div>
      {children} {/* children do NOT re-render when scrollY changes */}
    </div>
  );
}
```

## Common Pitfalls

1. **Checking children with typeof** — `children` can be a single element, an array, a string, or null. Use `React.Children.count()` or `React.Children.map()` if you need to inspect or manipulate children.
2. **Not typing children** — Forgetting to include `children: React.ReactNode` in your props type causes TypeScript errors when consumers try to nest content.

## Best Practices

1. **Use children for single-slot content injection** — When a component has one main content area, `children` is the natural choice. For multiple slots, use named props.
2. **Keep wrapper components thin** — A modal or layout component should handle only its structural concerns (overlay, spacing, close button) and delegate all content to children.

## Summary

- The `children` prop contains whatever JSX is placed between a component's opening and closing tags.
- Use `React.ReactNode` as the TypeScript type for children to accept all renderable content.
- Wrapper components that use children promote composition and keep your codebase flexible.

## Code Examples

**A wrapper component using children for flexible error messaging**

```tsx
function ErrorBoundaryFallback({ children }: { children: React.ReactNode }) {
  return (
    <div className="error-fallback" role="alert">
      <h2>Something went wrong</h2>
      {children}
    </div>
  );
}

// Usage
<ErrorBoundaryFallback>
  <p>We could not load your dashboard. Please try refreshing.</p>
  <button onClick={() => window.location.reload()}>Refresh</button>
</ErrorBoundaryFallback>
```


## Resources

- [Passing JSX as Children](https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children) — Official guide on the children prop

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*