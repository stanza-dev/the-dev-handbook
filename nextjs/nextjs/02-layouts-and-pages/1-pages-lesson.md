---
source_course: "nextjs"
source_lesson: "nextjs-layouts-and-pages-lesson"
---

# Pages and Layouts

## Pages

A **Page** is UI that is unique to a specific route. You define a page by exporting a component from a `page.js` file.

```tsx
// app/blog/page.tsx
// URL: /blog
export default function Page() {
  return <h1>Hello, Blog!</h1>
}
```

Pages are **Server Components** by default but can be set to Client Components.

## Layouts

A **Layout** is UI that is shared between multiple routes. On navigation, layouts preserve state, remain interactive, and do not re-render. Layouts can also be nested.

Define a layout by default exporting a React component from a `layout.js` file. The component should accept a `children` prop.

### Root Layout

The top-most layout is the **Root Layout** (`app/layout.js`). It is required and must contain `html` and `body` tags.

```tsx
export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```

## Code Examples

**A nested layout wrapping child pages**

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <section>
      <nav>Dashboard Nav</nav>
      {children}
    </section>
  )
}
```


## Resources

- [Layouts and Pages Docs](https://nextjs.org/docs/app/building-your-application/routing/layouts-and-pages) — Official guide to Layouts and Pages

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*