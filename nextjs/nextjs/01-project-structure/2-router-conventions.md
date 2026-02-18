---
source_course: "nextjs"
source_lesson: "nextjs-app-router-conventions"
---

# Project Structure

Next.js 13+ introduced the **App Router**, built on React Server Components. It uses a file-system based router where folders define routes.

## Special Files

Inside the `app/` directory, specific filenames define the behavior of deep segments:

*   `page.js`: The unique UI for a route and the point where the route becomes publicly accessible.
*   `layout.js`: Shared UI for a segment and its children. Preserves state on navigation.
*   `loading.js`: An optional Suspense boundary for the loading UI.
*   `error.js`: An optional Error boundary for the segment.
*   `not-found.js`: Specialized UI for 404 errors.
*   `route.js`: Server-side API endpoint handler.

## Component Hierarchy

React components defined in special files are rendered in a specific hierarchy:

`layout` > `template` > `error` > `loading` > `not-found` > `page`

## Colocation

You can colocate other files (components, styles, tests) inside the `app` directory. Only `page.js` or `route.js` make a route addressable.

## Code Examples

**Example file structure for nested routes**

```bash
app/
  layout.tsx      # Root layout
  page.tsx        # Home page (/)
  dashboard/
    layout.tsx    # Dashboard layout
    page.tsx      # Dashboard page (/dashboard)
    loading.tsx   # Loading UI for dashboard
```


## Resources

- [Project Structure Docs](https://nextjs.org/docs/getting-started/project-structure) — Full reference of Next.js file conventions

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*