---
source_course: "react-modern-patterns"
source_lesson: "react-server-client-boundary"
---

# The Server-Client Boundary

## Introduction

Understanding where the server ends and the client begins is the single most important concept when working with React Server Components. The boundary between Server and Client Components determines what code ships to the browser, how data flows through your application, and where interactivity lives. Getting this boundary right is the difference between a fast, efficient app and one that ships unnecessary JavaScript or fails with serialization errors.

## Key Concepts

- **Boundary**: The point in the component tree where a `"use client"` directive appears. Everything above it (without its own directive) is a Server Component; the marked component and its imports are Client Components.
- **Serialization**: Data crossing the boundary from server to client must be serializable to JSON. This means plain objects, arrays, strings, numbers, booleans, null, and a few special React types.
- **Children Pattern**: Server Components can pass other Server Components as `children` to Client Components because the server renders them first and sends the result.
- **Composition over Import**: Client Components cannot import Server Components, but they can render Server Components passed as children or props.

## Real World Context

Think of a dashboard layout where the sidebar needs a toggle animation (client-side) but the sidebar content includes user data fetched from the database (server-side). You would make the sidebar container a Client Component for the animation, but pass the server-rendered content as `children`. The database query stays on the server, the animation runs on the client, and no unnecessary data fetching code ships to the browser.

## Deep Dive

The children pattern is the most common way to compose Server and Client Components:

```tsx
// Sidebar.tsx — Client Component for animation
"use client";

import { useState } from "react";

export function Sidebar({ children }: { children: React.ReactNode }) {
  const [isOpen, setIsOpen] = useState(true);

  return (
    <aside className={isOpen ? "sidebar open" : "sidebar closed"}>
      <button onClick={() => setIsOpen(!isOpen)}>
        {isOpen ? "Collapse" : "Expand"}
      </button>
      {isOpen && children}
    </aside>
  );
}
```

```tsx
// DashboardLayout.tsx — Server Component
import { Sidebar } from "./Sidebar";
import { db } from "@/lib/database";

export default async function DashboardLayout({ children }: { children: React.ReactNode }) {
  const user = await db.user.findUnique({ where: { id: getCurrentUserId() } });
  const navItems = await db.navItem.findMany();

  return (
    <div className="dashboard">
      <Sidebar>
        {/* These are Server Components rendered as children */}
        <UserProfile user={user} />
        <NavMenu items={navItems} />
      </Sidebar>
      <main>{children}</main>
    </div>
  );
}
```

What you **cannot** do is import a Server Component inside a Client Component:

```tsx
// This WILL NOT work
"use client";

import { ServerOnlyWidget } from "./ServerOnlyWidget"; // Error!

function ClientThing() {
  return <ServerOnlyWidget />; // Cannot import Server Components in Client
}
```

Props crossing the boundary must be serializable:

```tsx
// Valid props across the boundary
<ClientComponent
  name="Alice"              // string
  count={42}                // number
  items={["a", "b"]}       // array
  config={{ key: "value" }} // plain object
/>

// Invalid props — will cause errors
<ClientComponent
  onClick={() => {}}        // function — not serializable
  date={new Date()}         // Date object — not serializable
  data={new Map()}          // Map — not serializable
/>
```

## Common Pitfalls

1. **Placing `"use client"` too high in the tree** — If you mark a layout component as a Client Component, every component it imports also becomes a Client Component. This can accidentally pull large libraries into the client bundle.
2. **Trying to pass functions from Server to Client** — Functions are not serializable. If you need a Client Component to call server-side logic, use Server Functions (`"use server"`) instead of passing callbacks.

## Best Practices

1. **Use the children pattern for composition** — Instead of importing Server Components in Client Components (which is not allowed), pass server-rendered content as `children` or named props like `header` and `footer`.
2. **Create thin Client Component wrappers** — When you need interactivity around server-rendered content, create a minimal Client Component that handles only the interactive behavior and accepts `children` for the server-rendered parts.

## Summary

- The `"use client"` directive creates a boundary; everything above stays on the server, the marked component and its imports go to the client.
- Data crossing the boundary must be serializable — plain objects, arrays, strings, numbers, and booleans are safe; functions and class instances are not.
- Use the children pattern to render Server Components inside Client Components without importing them directly.

## Code Examples

**Children pattern for composing Server and Client Components**

```tsx
// AnimatedContainer.tsx — Client Component (thin wrapper)
"use client";
import { useState } from "react";

export function AnimatedContainer({ children }: { children: React.ReactNode }) {
  const [visible, setVisible] = useState(true);
  return (
    <div>
      <button onClick={() => setVisible(!visible)}>Toggle</button>
      {visible && <div className="animate-fade-in">{children}</div>}
    </div>
  );
}

// Page.tsx — Server Component
import { AnimatedContainer } from "./AnimatedContainer";
import { db } from "@/lib/db";

export default async function Page() {
  const data = await db.items.findMany();
  return (
    <AnimatedContainer>
      {/* Server-rendered content passed as children */}
      {data.map(item => <p key={item.id}>{item.name}</p>)}
    </AnimatedContainer>
  );
}
```


## Resources

- [Composition Patterns](https://react.dev/reference/rsc/server-components#composing-server-and-client-components) — How to compose Server and Client Components together

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*