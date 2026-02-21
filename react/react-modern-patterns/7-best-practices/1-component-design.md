---
source_course: "react-modern-patterns"
source_lesson: "react-component-design"
---

# Component Design Principles

## Introduction

Well-designed React components are the foundation of a maintainable application. This lesson covers the core principles that guide component architecture: single responsibility, appropriate granularity, and effective prop design. Following these principles leads to components that are easy to understand, test, and reuse.

## Key Concepts

- **Single responsibility**: Each component should have one reason to change. If a component handles fetching, formatting, and displaying data, split it.
- **Composition over inheritance**: React components should be composed together, not extended via class inheritance.
- **Prop drilling**: Passing props through many intermediate components that do not use them. Solved by Context, composition, or state management libraries.

## Real World Context

A startup's codebase has a UserDashboard component with 500 lines: it fetches user data, manages sidebar state, renders notifications, displays charts, and handles settings. When the notification API changes, developers must understand the entire component to make a safe change. Splitting into focused components (UserHeader, NotificationsWidget, DashboardChart, QuickSettings) makes each piece independently understandable and testable.

## Deep Dive

**Single Responsibility in practice:**

```tsx
// Too many responsibilities
function UserDashboard() {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [notifications, setNotifications] = useState([]);
  // ... 200+ lines of mixed logic and JSX
}

// Composed of focused components
function UserDashboard() {
  return (
    <DashboardLayout>
      <UserHeader />
      <main>
        <PostsFeed />
        <Sidebar>
          <NotificationsWidget />
          <QuickSettings />
        </Sidebar>
      </main>
    </DashboardLayout>
  );
}
```

**Solving prop drilling with composition:**

```tsx
// Prop drilling through intermediaries
<App user={user}>
  <Layout user={user}>
    <Sidebar user={user}>
      <UserMenu user={user} />
    </Sidebar>
  </Layout>
</App>

// Composition: pass the rendered component instead
function App() {
  const user = useUser();
  return (
    <Layout sidebar={<UserMenu user={user} />}>
      <MainContent />
    </Layout>
  );
}
```

This eliminates the intermediary components needing to know about user at all.

**Prop design**: Keep prop interfaces narrow. A component that accepts 20 props is doing too much. If props naturally group together, they might indicate a missing abstraction.

## Common Pitfalls

1. **God components** — Components that handle multiple unrelated responsibilities become hard to maintain and test. Split them by responsibility.
2. **Premature abstraction** — Creating reusable components before you have multiple use cases leads to over-engineered APIs. Wait until you see duplication.

## Best Practices

1. **Split by responsibility, not by size** — A 100-line component that does one thing well is better than five 20-line components with tangled dependencies.
2. **Use composition to avoid prop drilling** — Pass rendered elements as props instead of raw data through intermediaries.

## Summary

- Apply single responsibility: each component should have one reason to change.
- Use composition (children, render props, slot props) to avoid prop drilling.
- Keep prop interfaces narrow and wait for duplication before abstracting.

## Code Examples

**Composition pattern to avoid prop drilling**

```tsx
// Composition: solving prop drilling
function App() {
  const user = useUser();
  return (
    <Layout sidebar={<UserMenu user={user} />}>
      <MainContent />
    </Layout>
  );
}

function Layout({ sidebar, children }: {
  sidebar: ReactNode;
  children: ReactNode;
}) {
  return (
    <div className="layout">
      <aside>{sidebar}</aside>
      <main>{children}</main>
    </div>
  );
}
```


## Resources

- [Thinking in React](https://react.dev/learn/thinking-in-react) — Official guide on breaking UI into components

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*