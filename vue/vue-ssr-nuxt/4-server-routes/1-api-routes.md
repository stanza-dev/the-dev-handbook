---
source_course: "vue-ssr-nuxt"
source_lesson: "vue-ssr-nuxt-api-routes"
---

# Creating API Routes

Nuxt includes Nitro server engine, allowing you to create API endpoints alongside your frontend.

## Basic API Route

```typescript
// server/api/hello.ts
export default defineEventHandler(() => {
  return { message: 'Hello from API!' }
})

// Accessible at: /api/hello
```

## HTTP Methods

```typescript
// server/api/users.get.ts - GET /api/users
export default defineEventHandler(() => {
  return [{ id: 1, name: 'Alice' }]
})

// server/api/users.post.ts - POST /api/users
export default defineEventHandler(async (event) => {
  const body = await readBody(event)
  // Create user...
  return { id: 2, ...body }
})

// server/api/users/[id].delete.ts - DELETE /api/users/:id
export default defineEventHandler((event) => {
  const id = getRouterParam(event, 'id')
  // Delete user...
  return { deleted: id }
})
```

## Reading Request Data

```typescript
// server/api/users.post.ts
export default defineEventHandler(async (event) => {
  // Request body
  const body = await readBody(event)
  
  // Query parameters (?page=1&limit=10)
  const query = getQuery(event)
  
  // Route parameters (/api/users/123)
  const id = getRouterParam(event, 'id')
  
  // Headers
  const authHeader = getHeader(event, 'authorization')
  
  // Cookies
  const token = getCookie(event, 'auth-token')
  
  return { body, query, id }
})
```

## Dynamic Routes

```typescript
// server/api/users/[id].ts
export default defineEventHandler((event) => {
  const id = getRouterParam(event, 'id')
  
  // Fetch user by ID
  const user = users.find(u => u.id === Number(id))
  
  if (!user) {
    throw createError({
      statusCode: 404,
      message: 'User not found'
    })
  }
  
  return user
})
```

## Error Handling

```typescript
// server/api/users/[id].ts
export default defineEventHandler((event) => {
  const id = getRouterParam(event, 'id')
  
  if (!id || isNaN(Number(id))) {
    throw createError({
      statusCode: 400,
      message: 'Invalid user ID'
    })
  }
  
  const user = findUser(id)
  
  if (!user) {
    throw createError({
      statusCode: 404,
      statusMessage: 'Not Found',
      message: `User ${id} not found`
    })
  }
  
  return user
})
```

## Response Handling

```typescript
export default defineEventHandler((event) => {
  // Set status code
  setResponseStatus(event, 201)
  
  // Set headers
  setHeader(event, 'Content-Type', 'application/json')
  setHeader(event, 'Cache-Control', 'max-age=3600')
  
  // Set cookie
  setCookie(event, 'token', 'abc123', {
    httpOnly: true,
    secure: true,
    maxAge: 60 * 60 * 24  // 1 day
  })
  
  return { success: true }
})
```

## Server Middleware

```typescript
// server/middleware/auth.ts
export default defineEventHandler((event) => {
  // Runs on every request
  const token = getHeader(event, 'authorization')
  
  if (event.path.startsWith('/api/protected')) {
    if (!token) {
      throw createError({
        statusCode: 401,
        message: 'Unauthorized'
      })
    }
    // Attach user to context
    event.context.user = verifyToken(token)
  }
})
```

## Server Utilities

```typescript
// server/utils/db.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

export { prisma }

// Auto-imported in server routes
// server/api/users.ts
export default defineEventHandler(async () => {
  return await prisma.user.findMany()
})
```

## Database Example

```typescript
// server/api/posts/index.get.ts
export default defineEventHandler(async () => {
  return await prisma.post.findMany({
    include: { author: true },
    orderBy: { createdAt: 'desc' }
  })
})

// server/api/posts/index.post.ts
export default defineEventHandler(async (event) => {
  const body = await readBody(event)
  
  const post = await prisma.post.create({
    data: {
      title: body.title,
      content: body.content,
      authorId: event.context.user.id
    }
  })
  
  return post
})

// server/api/posts/[id].put.ts
export default defineEventHandler(async (event) => {
  const id = getRouterParam(event, 'id')
  const body = await readBody(event)
  
  const post = await prisma.post.update({
    where: { id: Number(id) },
    data: body
  })
  
  return post
})
```

## Resources

- [Server Routes](https://nuxt.com/docs/guide/directory-structure/server) — Nuxt server directory

---

> 📘 *This lesson is part of the [Vue Server-Side Rendering & Nuxt](https://stanza.dev/courses/vue-ssr-nuxt) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*