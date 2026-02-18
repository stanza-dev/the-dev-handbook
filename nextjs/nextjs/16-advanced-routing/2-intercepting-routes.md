---
source_course: "nextjs"
source_lesson: "nextjs-intercepting-routes"
---

# Intercepting Routes

Intercepting routes allow you to load a route within the current layout while keeping the context of the current page. This is commonly used for **modals** (e.g., clicking a photo in a feed opens it in a modal).

## Convention

Use parentheses `( )` with dot notation:

*   `(.)` Match segments on the same level
*   `(..)` Match segments one level above
*   `(..)(..)` Match segments two levels above

When you click a link to an intercepted route, the modal renders. If you refresh the page (hard navigation), the full page renders instead.

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*