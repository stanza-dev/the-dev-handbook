---
source_course: "nextjs"
source_lesson: "nextjs-proxy-intro"
---

# Proxy (formerly Middleware)

In Next.js 16, the routing interception layer formerly known as "Middleware" has been renamed to **Proxy** (`proxy.ts` or `proxy.js`).

Proxy allows you to run code before a request is completed. You can modify the response by rewriting, redirecting, modifying the request or response headers, or responding directly.

## Usage

Create a `proxy.ts` file in the root of your project (or inside `src/` if you use it).

```ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

// This function can be marked `async` if using `await` inside
export function proxy(request: NextRequest) {
  return NextResponse.redirect(new URL('/home', request.url))
}

// See "Matching Paths" below to learn more
export const config = {
  matcher: '/about/:path*',
}
```

## Code Examples

**Conditional rewriting using Proxy**

```typescript
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
 
export function proxy(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.rewrite(new URL('/dashboard/user', request.url))
  }
}

```


---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*