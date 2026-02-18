---
source_course: "nextjs"
source_lesson: "nextjs-parallel-routes"
---

# Parallel Routes

Parallel Routes allow you to simultaneously render one or more pages within the same layout. They are useful for dashboards or splitting a view into independent slots.

## Slots

Parallel routes are created using **Slots**, defined with the `@folder` convention.

```tsx
app/
  layout.tsx
  page.tsx
  @analytics/page.tsx
  @team/page.tsx
```

Slots are passed as props to the shared layout:

```tsx
export default function Layout({ children, analytics, team }) {
  return (
    <>
      {children}
      {analytics}
      {team}
    </>
  )
}
```

## Resources

- [Parallel Routes Docs](https://nextjs.org/docs/app/building-your-application/routing/parallel-routes) — Official guide on Parallel Routes

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*