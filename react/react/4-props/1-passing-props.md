---
source_course: "react"
source_lesson: "react-passing-props"
---

# Passing Props to Components

## Introduction

Props (short for properties) are the mechanism React uses to pass data from parent components to child components. They are the primary way components communicate in a React application. Understanding props is fundamental because nearly every component you write will accept and use them.

## Key Concepts

- **Props Object**: Every component receives a single object argument containing all the properties passed to it.
- **Read-Only**: Props are immutable inside the receiving component. You must never modify them directly.
- **Destructuring**: The conventional way to access props is by destructuring the parameter in the function signature.
- **Default Values**: Use JavaScript default parameter syntax to provide fallback values for optional props.

## Real World Context

Consider a notification system. The parent component (a `NotificationList`) fetches notification data from an API and passes each notification's title, message, timestamp, and severity to a `NotificationCard` child component. The card does not fetch its own data — it simply renders what it receives through props. If the data changes, the parent re-renders with new props, and the card updates automatically.

## Deep Dive

### Passing Props

Pass props to a component just like HTML attributes:

```tsx
function App() {
  return (
    <UserCard
      name="Alice Chen"
      role="Engineer"
      avatarUrl="/avatars/alice.jpg"
      isOnline={true}
    />
  );
}
```

Strings can use quotes. Numbers, booleans, objects, arrays, and expressions use curly braces.

### Receiving Props with Destructuring

```tsx
type UserCardProps = {
  name: string;
  role: string;
  avatarUrl: string;
  isOnline: boolean;
};

function UserCard({ name, role, avatarUrl, isOnline }: UserCardProps) {
  return (
    <div className="user-card">
      <img src={avatarUrl} alt={name} />
      <h3>{name}</h3>
      <p>{role}</p>
      {isOnline && <span className="status-dot" />}
    </div>
  );
}
```

### Default Values

Use JavaScript's default parameter syntax for optional props:

```tsx
function Badge({ label, color = 'blue' }: { label: string; color?: string }) {
  return <span style={{ backgroundColor: color }}>{label}</span>;
}

// color defaults to 'blue'
<Badge label="New" />
// color is overridden to 'red'
<Badge label="Urgent" color="red" />
```

### Spreading Props

When forwarding many props to a child, you can use the spread operator:

```tsx
type InputProps = React.ComponentProps<'input'>;

function StyledInput(props: InputProps) {
  return <input className="styled-input" {...props} />;
}
```

## Common Pitfalls

1. **Mutating props directly** — Writing `props.name = 'Bob'` will cause errors or unpredictable behavior. Props are owned by the parent. If you need to change a value, use state.
2. **Passing unnecessary props** — Spreading an entire object into a component (`{...data}`) often passes props the child does not need, which can trigger unnecessary re-renders and makes the API unclear.

## Best Practices

1. **Always type your props** — Use a TypeScript type or interface for every component's props. Mark optional props with `?` and provide sensible defaults.
2. **Keep the props API minimal** — Pass only the data a component needs. Fewer props make components easier to understand, test, and reuse.

## Summary

- Props are read-only data passed from parent to child components via JSX attributes.
- Destructure props in the function signature and use TypeScript types for safety.
- Use default parameter values for optional props rather than conditional logic inside the component.

## Code Examples

**A typed Card component with an optional variant prop and default value**

```tsx
type CardProps = {
  title: string;
  children: React.ReactNode;
  variant?: 'default' | 'outlined';
};

function Card({ title, children, variant = 'default' }: CardProps) {
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


## Resources

- [Passing Props to a Component](https://react.dev/learn/passing-props-to-a-component) — Official guide on passing and receiving props

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*