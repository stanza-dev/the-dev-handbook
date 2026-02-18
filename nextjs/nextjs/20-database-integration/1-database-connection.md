---
source_course: "nextjs"
source_lesson: "nextjs-database-connection"
---

# Database Integration

In Next.js, you can connect to your database directly inside **Server Components** or **Server Actions**. You do not need to create separate API routes for internal data fetching.

## Best Practices

*   **Server-only**: Ensure your database logic runs only on the server.
*   **Caching**: Database queries in Server Components are not automatically cached (unlike fetch), so consider using `unstable_cache` or the `use cache` directive (Next.js 16) if needed.

Common ORMs include Prisma and Drizzle.

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*