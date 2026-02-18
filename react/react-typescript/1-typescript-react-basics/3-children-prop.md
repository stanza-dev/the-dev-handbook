---
source_course: "react-typescript"
source_lesson: "react-typescript-children-prop"
---

# Typing the Children Prop

The `children` prop requires special attention in TypeScript.

## React.ReactNode (Most Common)

Accepts anything React can render:

```tsx
type CardProps = {
  title: string;
  children: React.ReactNode;
};

function Card({ title, children }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-body">{children}</div>
    </div>
  );
}

// All valid:
<Card title="Hello">Text content</Card>
<Card title="Hello"><span>Element</span></Card>
<Card title="Hello">{null}</Card>
<Card title="Hello">{items.map(i => <Item key={i.id} />)}</Card>
```

## React.ReactElement (Stricter)

Only accepts JSX elements:

```tsx
type LayoutProps = {
  header: React.ReactElement;
  children: React.ReactElement;
};

function Layout({ header, children }: LayoutProps) {
  return (
    <div>
      <header>{header}</header>
      <main>{children}</main>
    </div>
  );
}

// Valid:
<Layout header={<Header />}><Content /></Layout>

// Error:
<Layout header="Just text"><Content /></Layout>
```

## PropsWithChildren Helper

```tsx
import { PropsWithChildren } from 'react';

type CardProps = PropsWithChildren<{
  title: string;
}>;

// Equivalent to:
type CardProps = {
  title: string;
  children?: React.ReactNode;
};
```

## Specific Child Types

```tsx
// Only accept specific element types
type TabsProps = {
  children: React.ReactElement<TabProps> | React.ReactElement<TabProps>[];
};

// Function as children (render props)
type DataProviderProps = {
  children: (data: User[]) => React.ReactNode;
};

function DataProvider({ children }: DataProviderProps) {
  const data = useData();
  return <>{children(data)}</>;
}

// Usage:
<DataProvider>
  {(users) => users.map(u => <UserCard key={u.id} user={u} />)}
</DataProvider>
```

## No Children

For components that shouldn't have children:

```tsx
type IconProps = {
  name: string;
  size?: number;
  // Don't include children
};

function Icon({ name, size = 24 }: IconProps) {
  return <svg>...</svg>;
}

// TypeScript error if you try:
<Icon name="star">Text</Icon>
```

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*