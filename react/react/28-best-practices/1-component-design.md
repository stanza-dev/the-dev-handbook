---
source_course: "react"
source_lesson: "react-component-design"
---

# Component Design Best Practices

## Single Responsibility

Each component should do one thing well.

## Keep Components Small

If a component grows too large, split it.

## Lift State Up Sparingly

Only lift state when necessary. Keep state as local as possible.

## Avoid Prop Drilling

Use Context or composition for deeply nested data.

## Prefer Composition Over Inheritance

React components should be composed, not extended.

## Code Examples

**Component design patterns**

```tsx
// ❌ Bad: Component doing too much
function UserDashboard() {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [notifications, setNotifications] = useState([]);
  const [settings, setSettings] = useState({});
  // ... lots of logic
  
  return (
    <div>
      {/* 200+ lines of JSX */}
    </div>
  );
}

// ✅ Good: Composed of focused components
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

// ❌ Bad: Prop drilling through many levels
<App user={user}>
  <Layout user={user}>
    <Sidebar user={user}>
      <UserMenu user={user} />
    </Sidebar>
  </Layout>
</App>

// ✅ Good: Use Context for global data
const UserContext = createContext(null);

function App() {
  const [user, setUser] = useState(null);
  
  return (
    <UserContext.Provider value={user}>
      <Layout>
        <Sidebar>
          <UserMenu /> {/* Uses useContext(UserContext) */}
        </Sidebar>
      </Layout>
    </UserContext.Provider>
  );
}

// ✅ Good: Composition to avoid prop drilling
function App() {
  const [user, setUser] = useState(null);
  
  return (
    <Layout
      sidebar={<UserMenu user={user} />}
    >
      <MainContent />
    </Layout>
  );
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*