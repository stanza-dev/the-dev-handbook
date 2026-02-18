---
source_course: "react-server-components"
source_lesson: "react-server-components-composition"
---

# Composing Server and Client Components

The key to RSC architecture is understanding how Server and Client Components can work together.

## The Golden Rule

**Client Components cannot import Server Components directly.**

This would pull server code into the client bundle:

```tsx
'use client';

// ❌ This doesn't work!
import ServerComponent from './ServerComponent';

function ClientComponent() {
  return <ServerComponent />; // Error!
}
```

## The Solution: Children and Props

Pass Server Components as `children` or props to Client Components:

```tsx
// ServerParent.tsx (Server Component)
import ClientWrapper from './ClientWrapper';
import ServerContent from './ServerContent';

function ServerParent() {
  return (
    <ClientWrapper>
      <ServerContent /> {/* ✅ This works! */}
    </ClientWrapper>
  );
}
```

```tsx
// ClientWrapper.tsx
'use client';

function ClientWrapper({ children }) {
  const [isOpen, setIsOpen] = useState(true);
  
  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      {isOpen && children} {/* Server Component renders here */}
    </div>
  );
}
```

## Why This Works

When React renders on the server:

1. `ServerParent` renders, including `ServerContent`
2. `ServerContent` becomes serialized React elements (not code)
3. `ClientWrapper` receives these elements as `children`
4. The client only gets `ClientWrapper`'s code + the rendered output

## Practical Patterns

### Pattern 1: Interactive Wrapper

```tsx
// Server Component
async function ProductList() {
  const products = await getProducts();
  
  return (
    <Carousel> {/* Client Component */}
      {products.map(p => (
        <ProductCard key={p.id} product={p} /> {/* Server Component */}
      ))}
    </Carousel>
  );
}
```

### Pattern 2: Slot Pattern

```tsx
// Server Component
function Dashboard() {
  return (
    <DashboardLayout
      sidebar={<Sidebar />}      {/* Server Component */}
      header={<Header />}        {/* Server Component */}
      main={<Analytics />}       {/* Server Component */}
    />
  );
}

// Client Component
'use client';

function DashboardLayout({ sidebar, header, main }) {
  const [sidebarOpen, setSidebarOpen] = useState(true);
  
  return (
    <div className="dashboard">
      {sidebarOpen && <aside>{sidebar}</aside>}
      <div className="main-area">
        <header>{header}</header>
        <main>{main}</main>
      </div>
    </div>
  );
}
```

### Pattern 3: Context Providers

Providers must be Client Components, but their children can be Server Components:

```tsx
// Providers.tsx
'use client';

export function Providers({ children }) {
  return (
    <ThemeProvider>
      <AuthProvider>
        {children}
      </AuthProvider>
    </ThemeProvider>
  );
}

// layout.tsx (Server Component)
import { Providers } from './Providers';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Providers>
          {children} {/* Server Components work here! */}
        </Providers>
      </body>
    </html>
  );
}
```

## Resources

- [Composing Server and Client Components](https://react.dev/reference/rsc/server-components#composing-server-and-client-components) — Official React guide on composing Server and Client Components

---

> 📘 *This lesson is part of the [React Server Components](https://stanza.dev/courses/react-server-components) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*