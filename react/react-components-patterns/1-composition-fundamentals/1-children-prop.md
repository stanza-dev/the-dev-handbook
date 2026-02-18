---
source_course: "react-components-patterns"
source_lesson: "react-components-patterns-children-prop"
---

# The Children Prop: Containment Pattern

The `children` prop is React's primary mechanism for component composition. It allows components to accept arbitrary nested content.

## Basic Containment

Some components don't know their children ahead of time. This is common for generic "container" components:

```tsx
function Card({ children }) {
  return (
    <div className="card">
      <div className="card-content">
        {children}
      </div>
    </div>
  );
}

// Usage
<Card>
  <h2>Welcome</h2>
  <p>This is the card content.</p>
  <Button>Learn More</Button>
</Card>
```

## What Can Children Be?

`children` can be any valid React node:

```tsx
// String
<Card>Hello World</Card>

// Element
<Card><Button /></Card>

// Multiple elements
<Card>
  <Header />
  <Content />
</Card>

// Array
<Card>{items.map(item => <Item key={item.id} {...item} />)}</Card>

// Expression
<Card>{isLoggedIn ? <Dashboard /> : <Login />}</Card>

// Nothing
<Card>{null}</Card>
<Card>{false}</Card>
```

## TypeScript Typing

```tsx
import { ReactNode, PropsWithChildren } from 'react';

// Option 1: Explicit ReactNode
type CardProps = {
  title: string;
  children: ReactNode;
};

function Card({ title, children }: CardProps) {
  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  );
}

// Option 2: PropsWithChildren helper
type CardProps = PropsWithChildren<{
  title: string;
}>;

function Card({ title, children }: CardProps) {
  // ...
}
```

## Conditional Children

```tsx
function Collapsible({ title, children }) {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>
        {title} {isOpen ? '▼' : '▶'}
      </button>
      {isOpen && (
        <div className="content">
          {children}
        </div>
      )}
    </div>
  );
}
```

## Wrapping Children

```tsx
function FadeIn({ children, delay = 0 }) {
  return (
    <div 
      className="fade-in"
      style={{ animationDelay: `${delay}ms` }}
    >
      {children}
    </div>
  );
}

// Usage
<FadeIn delay={200}>
  <Card>Content appears with animation</Card>
</FadeIn>
```

## When to Use Children

✅ **Good use cases:**
- Layout components (Card, Modal, Sidebar)
- Wrapper components (ErrorBoundary, Provider)
- Generic containers (List, Grid)

❌ **Consider alternatives when:**
- You need multiple "slots" (use named props)
- Children need specific props injected (use render props)
- You need to validate children types (use compound components)

## Resources

- [Passing JSX as Children](https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children) — Official React documentation on the children prop

---

> 📘 *This lesson is part of the [React Components & Patterns](https://stanza.dev/courses/react-components-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*