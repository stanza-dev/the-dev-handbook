---
source_course: "nextjs"
source_lesson: "nextjs-css-support"
---

# CSS in Next.js

Next.js supports multiple ways to style your application.

## Global CSS

Import global styles (like `globals.css`) in your **Root Layout** (`app/layout.tsx`).

```tsx
import './globals.css'
```

## CSS Modules

Next.js has built-in support for CSS Modules using the `.module.css` extension. CSS Modules locally scope CSS by automatically creating a unique class name.

```tsx
import styles from './styles.module.css'

export default function Page() {
  return <div className={styles.dashboard}>Dashboard</div>
}
```

## Tailwind CSS

Next.js has built-in support for Tailwind CSS. Install it and configure your `globals.css` with the Tailwind directives.

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*