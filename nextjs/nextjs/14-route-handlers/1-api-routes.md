---
source_course: "nextjs"
source_lesson: "nextjs-api-routes"
---

# Route Handlers

Route Handlers allows you to create custom request handlers for a given route using the Web Request and Response APIs.

## Usage

Route Handlers are defined in a `route.js|ts` file inside the `app` directory.

```tsx
// app/api/hello/route.ts
import { NextResponse } from 'next/server'

export async function GET(request: Request) {
  return NextResponse.json({ message: 'Hello World' })
}
```

Supported HTTP methods: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, and `OPTIONS`. If an unsupported method is called, Next.js will return a 405 Method Not Allowed response.

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*