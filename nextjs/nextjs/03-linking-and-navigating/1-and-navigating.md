---
source_course: "nextjs"
source_lesson: "nextjs-linking-and-navigating"
---

# Linking and Navigating

In Next.js, routes are rendered on the server by default. This means the client often waits for a server response before showing a new route. Next.js improves this with **prefetching**, **streaming**, and **client-side transitions**.

## The `Link` Component

The primary way to navigate between routes is the `<Link>` component. It extends the HTML `<a>` element to provide prefetching and client-side navigation.

Import it from `next/link`:

```tsx
import Link from 'next/link'

export default function Page() {
  return <Link href="/dashboard">Dashboard</Link>
}
```

## Prefetching

Prefetching loads a route in the background before the user visits it. When a `<Link>` enters the user's viewport, Next.js automatically prefetches the linked route (in production). This makes navigation feel instant.

## Code Examples

**Basic navigation with the Link component**

```tsx
import Link from 'next/link'

export default function NavBar() {
  return (
    <nav>
      <Link href="/">Home</Link>
      <Link href="/about">About</Link>
    </nav>
  )
}
```


## Resources

- [Next.js Link API Reference](https://nextjs.org/docs/app/api-reference/components/link) — Official documentation for the Link component

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*