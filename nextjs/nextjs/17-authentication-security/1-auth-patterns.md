---
source_course: "nextjs"
source_lesson: "nextjs-auth-patterns"
---

# Authentication Patterns

Authentication in Next.js relies on standard web patterns like Cookies and Sessions. Major solutions include:

1.  **Auth.js (NextAuth)**: A complete open-source authentication solution.
2.  **Server Actions**: Implement custom auth flows using cookies and server actions.

## Protecting Content

Since Server Components run on the server, you can check for valid sessions directly in your component or layout.

```tsx
import { redirect } from 'next/navigation'
import { getSession } from './lib/auth'

export default async function Dashboard() {
  const session = await getSession()
  if (!session) redirect('/login')
  return <div>Welcome {session.user}</div>
}
```

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*