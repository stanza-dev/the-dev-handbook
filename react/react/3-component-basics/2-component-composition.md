---
source_course: "react"
source_lesson: "react-component-composition"
---

# Component Composition

## Introduction

Composition is the primary pattern for building complex UIs in React. Instead of creating one massive component, you break the UI into small, focused pieces and combine them like building blocks. This lesson teaches you how to think in terms of composition and how to structure component hierarchies effectively.

## Key Concepts

- **Composition over Inheritance**: React favors composing components together rather than extending base components with inheritance.
- **Container and Presentational Components**: Containers manage data and logic; presentational components focus on how things look.
- **Slot Pattern**: Using `children` or named props to let parent components inject content into specific locations.
- **Component Hierarchy**: The tree of parent-child relationships that forms your application structure.

## Real World Context

Consider a settings page with multiple sections: Profile, Notifications, and Billing. Each section has a card layout with a title, description, and action area. Instead of duplicating that card layout three times, you create a reusable `SettingsCard` component that accepts `title`, `description`, and `children` props. Each section becomes a thin wrapper that fills in its specific content.

## Deep Dive

### Basic Composition

The simplest form of composition is rendering one component inside another:

```tsx
function Avatar({ src, alt }: { src: string; alt: string }) {
  return <img className="avatar" src={src} alt={alt} />;
}

function UserCard({ user }: { user: { name: string; avatarUrl: string } }) {
  return (
    <div className="user-card">
      <Avatar src={user.avatarUrl} alt={user.name} />
      <h3>{user.name}</h3>
    </div>
  );
}
```

### The Slot Pattern with Children

Use `children` to create flexible wrapper components:

```tsx
function Card({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <div className="card">
      <h2 className="card-title">{title}</h2>
      <div className="card-body">{children}</div>
    </div>
  );
}

// Different content for the same card layout
function App() {
  return (
    <>
      <Card title="Profile">
        <p>Edit your name and avatar.</p>
        <button>Update Profile</button>
      </Card>
      <Card title="Billing">
        <p>Manage your subscription.</p>
        <button>View Plans</button>
      </Card>
    </>
  );
}
```

### Multiple Slots with Named Props

When you need more than one injection point, use named props:

```tsx
type PageLayoutProps = {
  header: React.ReactNode;
  sidebar: React.ReactNode;
  children: React.ReactNode;
};

function PageLayout({ header, sidebar, children }: PageLayoutProps) {
  return (
    <div className="page">
      <div className="page-header">{header}</div>
      <div className="page-body">
        <aside className="page-sidebar">{sidebar}</aside>
        <main className="page-content">{children}</main>
      </div>
    </div>
  );
}
```

### Extracting Components

When a component grows beyond 50-80 lines or handles multiple concerns, split it. Look for natural boundaries: a form field, a list item, a navigation group.

## Common Pitfalls

1. **Over-abstracting too early** — Do not create reusable components before you have at least two concrete use cases. Premature abstraction leads to inflexible APIs.
2. **Prop drilling** — Passing props through many intermediate components that do not use them is a code smell. Consider Context or component composition (passing components as props) to solve this.

## Best Practices

1. **Prefer composition over configuration props** — Instead of a component with 15 boolean configuration props, let the parent compose the exact UI it needs using children and slots.
2. **Keep components focused** — Each component should have a single responsibility. If you find yourself naming a component `UserCardWithEditButtonAndAvatar`, it is time to split it up.

## Summary

- Composition is how you build complex UIs from simple, focused components in React.
- Use `children` for single-slot layouts and named props for multi-slot layouts.
- Extract new components when an existing one grows too large or handles multiple concerns.

## Code Examples

**A reusable SettingsCard composed with different content via children**

```tsx
type SettingsCardProps = {
  title: string;
  description: string;
  children: React.ReactNode;
};

function SettingsCard({ title, description, children }: SettingsCardProps) {
  return (
    <div className="settings-card">
      <div className="settings-card-header">
        <h3>{title}</h3>
        <p>{description}</p>
      </div>
      <div className="settings-card-actions">{children}</div>
    </div>
  );
}

function SettingsPage() {
  return (
    <div>
      <SettingsCard title="Notifications" description="Choose what you get notified about.">
        <label><input type="checkbox" /> Email alerts</label>
        <label><input type="checkbox" /> Push notifications</label>
      </SettingsCard>
      <SettingsCard title="Danger Zone" description="Irreversible actions.">
        <button className="btn-danger">Delete Account</button>
      </SettingsCard>
    </div>
  );
}
```


## Resources

- [Passing JSX as Children](https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children) — Official guide on the children prop pattern

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*