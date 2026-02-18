---
source_course: "nextjs"
source_lesson: "nextjs-server-actions"
---

# Updating Data with Server Actions

**Server Actions** are asynchronous functions that run on the server. They are reusable and can be invoked from Client Components or Server Components.

## Creating a Server Action

To define a server action, add the `'use server'` directive at the top of an async function.

```tsx
export default function Page() {
  async function create(formData: FormData) {
    'use server'
    // Mutate data here
  }

  return <form action={create}>...</form>
}
```

Server Actions integrate seamlessly with the HTML `form` element via the `action` prop, progressively enhancing forms.

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*