---
source_course: "nextjs"
source_lesson: "nextjs-next-font-google"
---

# Font Optimization

`next/font` automatically optimizes your fonts (including custom fonts) and removes external network requests for improved privacy and performance.

## Google Fonts

Automatically self-host any Google Font. Fonts are included in the deployment and served from the same domain.

```tsx
import { Inter } from 'next/font/google'

const inter = Inter({ subsets: ['latin'] })

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={inter.className}>
      <body>{children}</body>
    </html>
  )
}
```

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*