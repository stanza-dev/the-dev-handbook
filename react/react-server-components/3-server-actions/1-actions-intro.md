---
source_course: "react-server-components"
source_lesson: "react-server-components-server-actions-intro"
---

# Server Actions: Server-Side Mutations

Server Actions are async functions that run on the server. They're marked with `'use server'` and can be called from Client Components to perform mutations.

## The Basic Pattern

```tsx
// actions.ts
'use server';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;
  
  await db.posts.create({
    data: { title, content }
  });
  
  revalidatePath('/posts');
}
```

```tsx
// PostForm.tsx
'use client';

import { createPost } from './actions';

export function PostForm() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="Title" required />
      <textarea name="content" placeholder="Content" required />
      <button type="submit">Create Post</button>
    </form>
  );
}
```

## Why Server Actions?

### Before: Manual API Routes

```tsx
// pages/api/posts.ts
export default async function handler(req, res) {
  if (req.method === 'POST') {
    const { title, content } = req.body;
    await db.posts.create({ data: { title, content } });
    res.status(200).json({ success: true });
  }
}

// Component
'use client';
function PostForm() {
  const handleSubmit = async (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    await fetch('/api/posts', {
      method: 'POST',
      body: JSON.stringify(Object.fromEntries(formData))
    });
  };
  
  return <form onSubmit={handleSubmit}>...</form>;
}
```

### After: Server Actions

```tsx
// actions.ts
'use server';

export async function createPost(formData: FormData) {
  await db.posts.create({
    data: {
      title: formData.get('title'),
      content: formData.get('content')
    }
  });
}

// Component
'use client';
import { createPost } from './actions';

function PostForm() {
  return (
    <form action={createPost}>
      <input name="title" />
      <textarea name="content" />
      <button type="submit">Create</button>
    </form>
  );
}
```

## Two Ways to Define Server Actions

### 1. Module-Level Directive

```tsx
// actions.ts
'use server';

// All exports are Server Actions
export async function createUser(data) { ... }
export async function deleteUser(id) { ... }
export async function updateUser(id, data) { ... }
```

### 2. Inline in Server Components

```tsx
// ServerComponent.tsx (Server Component)
async function ServerComponent() {
  async function handleSubmit(formData: FormData) {
    'use server';
    // This function runs on the server
    await db.items.create({ ... });
  }
  
  return (
    <form action={handleSubmit}>
      <input name="item" />
      <button>Add</button>
    </form>
  );
}
```

## Progressive Enhancement

Forms with Server Actions work even before JavaScript loads:

```tsx
<form action={serverAction}>
  {/* Works without JS! */}
  <input name="email" type="email" />
  <button>Subscribe</button>
</form>
```

This is because the form submits as a regular HTML form, and the server handles it.

## Resources

- [Server Actions](https://react.dev/reference/rsc/server-functions) — Official React documentation for Server Functions (Actions)
- ['use server' Directive](https://react.dev/reference/rsc/use-server) — Official React documentation for the 'use server' directive

---

> 📘 *This lesson is part of the [React Server Components](https://stanza.dev/courses/react-server-components) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*