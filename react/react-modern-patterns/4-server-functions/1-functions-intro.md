---
source_course: "react-modern-patterns"
source_lesson: "react-server-functions-intro"
---

# Server Functions (Server Actions)

## Introduction

Server Functions (also known as Server Actions) let you define async functions that execute exclusively on the server but can be called seamlessly from Client Components. They eliminate the need for manually creating API routes for data mutations. You mark a function with `"use server"` and React handles the serialization, network transport, and response — all you write is a function.

## Key Concepts

- **`"use server"` directive**: Marks a function or module to run on the server. When used at the top of a file, all exported functions become Server Functions.
- **Inline directive**: You can place `"use server"` inside an async function body within a Server Component to make just that function server-only.
- **Automatic RPC**: React serializes the arguments, sends them to the server via a POST request, executes the function, and sends the result back — transparent to the developer.
- **Integration with forms**: Server Functions can be passed directly to a form's `action` prop, receiving FormData as an argument.

## Real World Context

Consider a blog application where users can create, edit, and delete posts. Without Server Functions, you would need to create three separate API routes (`POST /api/posts`, `PUT /api/posts/:id`, `DELETE /api/posts/:id`), write fetch calls on the client, handle request/response serialization, and manage CSRF tokens. With Server Functions, you write one file with three exported async functions, mark it `"use server"`, and call them directly from your Client Components.

## Deep Dive

There are two ways to define Server Functions. The first is at the module level:

```tsx
// actions.ts
"use server";

import { db } from "@/lib/db";
import { revalidatePath } from "next/cache";

export async function createPost(formData: FormData) {
  const title = formData.get("title") as string;
  const content = formData.get("content") as string;

  await db.post.create({ data: { title, content } });
  revalidatePath("/posts");
}

export async function deletePost(postId: string) {
  await db.post.delete({ where: { id: postId } });
  revalidatePath("/posts");
}
```

The second is inline within a Server Component:

```tsx
// Page.tsx (Server Component)
export default async function Page() {
  async function handleLike() {
    "use server";
    await db.like.create({ data: { postId: "123" } });
    revalidatePath("/posts");
  }

  return (
    <form action={handleLike}>
      <button type="submit">Like</button>
    </form>
  );
}
```

You can call Server Functions from Client Components using `useActionState` or directly:

```tsx
"use client";

import { useActionState } from "react";
import { createPost } from "./actions";

export function CreatePostForm() {
  const [state, formAction, isPending] = useActionState(
    async (prevState, formData) => {
      const result = await createPost(formData);
      return result;
    },
    null
  );

  return (
    <form action={formAction}>
      <input name="title" required />
      <textarea name="content" required />
      <button disabled={isPending}>
        {isPending ? "Creating..." : "Create Post"}
      </button>
    </form>
  );
}
```

## Common Pitfalls

1. **Forgetting that Server Functions are public endpoints** — Any function marked `"use server"` becomes callable from the client. Always validate inputs and authenticate the user inside the function, never trust that the caller is authorized.
2. **Trying to use `"use server"` in a Client Component file** — The inline `"use server"` directive only works inside Server Components. In a `"use client"` file, you must import Server Functions from a separate `"use server"` module.

## Best Practices

1. **Create a dedicated `actions.ts` file** — Colocate related Server Functions in a single file with the `"use server"` directive at the top. This makes them easy to find, test, and secure.
2. **Always validate and authorize** — Treat every Server Function as a public API endpoint. Validate all inputs with a schema library like Zod and check that the current user has permission to perform the operation.

## Summary

- Server Functions are async functions marked with `"use server"` that run on the server but can be called from Client Components.
- They replace traditional API routes for data mutations, handling serialization and network transport automatically.
- Always validate inputs and authenticate users inside Server Functions because they are publicly callable endpoints.

## Code Examples

**Server Function with form integration and useActionState**

```tsx
// actions.ts
"use server";

import { db } from "@/lib/db";
import { revalidatePath } from "next/cache";

export async function createComment(prevState: any, formData: FormData) {
  const postId = formData.get("postId") as string;
  const content = formData.get("content") as string;

  if (!content || content.length < 3) {
    return { error: "Comment must be at least 3 characters" };
  }

  await db.comment.create({ data: { postId, content } });
  revalidatePath("/posts/" + postId);
  return { error: null };
}

// CommentForm.tsx
"use client";
import { useActionState } from "react";
import { createComment } from "./actions";

export function CommentForm({ postId }: { postId: string }) {
  const [state, formAction, isPending] = useActionState(createComment, null);
  return (
    <form action={formAction}>
      <input type="hidden" name="postId" value={postId} />
      <textarea name="content" disabled={isPending} />
      <button disabled={isPending}>{isPending ? "Posting..." : "Post"}</button>
      {state?.error && <p className="error">{state.error}</p>}
    </form>
  );
}
```


## Resources

- [Server Functions](https://react.dev/reference/rsc/server-functions) — Official Server Functions guide

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*