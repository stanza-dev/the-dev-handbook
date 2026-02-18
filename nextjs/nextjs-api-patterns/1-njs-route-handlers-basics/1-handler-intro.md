---
source_course: "nextjs-api-patterns"
source_lesson: "nextjs-api-patterns-njs-route-handler-intro"
---

# Introduction to Route Handlers

## Introduction

Every modern web application needs an API. Whether you're fetching data for your frontend, handling form submissions, or building a public API, Route Handlers are how you create API endpoints in Next.js.

## Key Concepts

**Route Handlers** are API endpoints built using the Web Platform's `Request` and `Response` APIs:

- **File-based routing**: Create `route.ts` in any folder to define an endpoint
- **HTTP methods**: Export functions named `GET`, `POST`, `PUT`, `DELETE`, etc.
- **Server-only**: Route Handlers run exclusively on the server
- **Colocation**: Place them anywhere in the `app/` directory

## Real World Context

Route Handlers power:
- Backend APIs consumed by your React frontend
- Webhooks from third-party services (Stripe, GitHub)
- Form submission handlers
- Public REST APIs for external consumers
- Server-side data processing

## Deep Dive

### Creating Your First Route Handler

Create `app/api/hello/route.ts`:

```typescript
// app/api/hello/route.ts
export async function GET() {
  return Response.json({ message: 'Hello, World!' });
}
```

This creates an endpoint at `GET /api/hello`.

### Supported HTTP Methods

```typescript
// All supported methods
export async function GET(request: Request) { ... }
export async function POST(request: Request) { ... }
export async function PUT(request: Request) { ... }
export async function PATCH(request: Request) { ... }
export async function DELETE(request: Request) { ... }
export async function HEAD(request: Request) { ... }
export async function OPTIONS(request: Request) { ... }
```

### File Structure Examples

```
app/
  api/
    users/
      route.ts          → GET/POST /api/users
      [id]/
        route.ts        → GET/PUT/DELETE /api/users/:id
    posts/
      route.ts          → GET/POST /api/posts
  webhooks/
    stripe/
      route.ts          → POST /webhooks/stripe
```

### Important Constraint

`route.ts` and `page.tsx` **cannot coexist** in the same folder:

```
❌ app/api/route.ts + app/api/page.tsx   // Error!
✅ app/api/route.ts                      // OK
✅ app/api/docs/page.tsx                 // OK (different folder)
```

## Common Pitfalls

1. **Mixing route.ts and page.tsx**: They can't be in the same directory. Choose one or use a subdirectory.

2. **Wrong export name**: Function must be named exactly `GET`, `POST`, etc. (uppercase). `get()` or `getHandler()` won't work.

3. **Forgetting async**: While not always required, Route Handlers typically perform async operations. Always use `async function`.

## Best Practices

- **Use the `/api` convention**: Keep APIs under `/api` for clarity (though not required)
- **One resource per file**: Group related methods (GET, POST) but keep different resources separate
- **Use TypeScript**: Type your request/response for better developer experience
- **Return appropriate status codes**: Don't just return 200 for everything

## Summary

Route Handlers are Next.js's way of building API endpoints using standard Web APIs. Create a `route.ts` file, export functions named after HTTP methods, and you have an API. They're server-only, file-based, and can be placed anywhere in your `app/` directory.

## Resources

- [Route Handlers Documentation](https://nextjs.org/docs/app/building-your-application/routing/route-handlers) — Official Route Handlers guide

---

> 📘 *This lesson is part of the [Next.js API Routes & Patterns](https://stanza.dev/courses/nextjs-api-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*