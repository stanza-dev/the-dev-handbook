---
source_course: "nextjs"
source_lesson: "nextjs-zod-validation"
---

# Forms & Validation

When using Server Actions for form mutations, it is critical to validate data before processing it. **Zod** is a popular schema validation library for this purpose.

## Example

```ts
'use server'

import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
})

export async function createUser(formData: FormData) {
  const parsed = schema.safeParse({
    email: formData.get('email'),
  })

  if (!parsed.success) {
    return { errors: parsed.error.flatten().fieldErrors }
  }

  // Proceed with safe data
}
```

---

> 📘 *This lesson is part of the [Next.js Full-Stack](https://stanza.dev/courses/nextjs) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*