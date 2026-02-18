---
source_course: "react"
source_lesson: "react-passing-props"
---

# Props (Properties)

Props are how you pass data from parent to child components. They're read-only!

## Passing Props

Pass props like HTML attributes:
```jsx
<Avatar name="Alice" size={100} />
```

## Receiving Props

Access props via the function parameter:
```jsx
function Avatar({ name, size }) {
  // use name and size
}
```

## Default Props

Use destructuring defaults:
```jsx
function Avatar({ size = 50 }) { ... }
```

## Code Examples

**TypeScript component with typed props**

```tsx
type CardProps = {
  title: string;
  children: React.ReactNode;
  variant?: 'default' | 'outlined';
};

function Card({ 
  title, 
  children, 
  variant = 'default' 
}: CardProps) {
  return (
    <div className={`card card--${variant}`}>
      <h3>{title}</h3>
      <div>{children}</div>
    </div>
  );
}

// Usage
<Card title="Welcome" variant="outlined">
  <p>Hello, World!</p>
</Card>
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*