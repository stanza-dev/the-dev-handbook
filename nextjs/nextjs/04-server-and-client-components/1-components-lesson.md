---
source_course: "nextjs"
source_lesson: "nextjs-server-and-client-components-lesson"
---

# Server and Client Components

Next.js allow you to write React that renders on the server or the client.

## Server Components (Default)

All components in the App Router are **Server Components** by default. They run only on the server and are never downloaded by the client. This reduces bundle size and allows secure access to backend resources.

## Client Components

Client Components run in the browser (and are pre-rendered on the server). To use them, add the `'use client'` directive at the top of your file.

Use Client Components when you need:
*   Interactivity and event listeners (`onClick`, `onChange`)
*   State and Lifecycle Effects (`useState`, `useEffect`)
*   Browser APIs

```tsx
'use client'

import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*