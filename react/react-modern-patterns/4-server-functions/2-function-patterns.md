---
source_course: "react-modern-patterns"
source_lesson: "react-server-function-patterns"
---

# Server Function Patterns

## Introduction

Now that you understand what Server Functions are and how they work, let us explore the patterns that make them powerful in production applications. From combining Server Functions with `useOptimistic` for instant feedback to handling complex multi-step mutations, these patterns will help you build robust, user-friendly applications that bridge the server-client divide elegantly.

## Key Concepts

- **Optimistic + Server Function**: Trigger an optimistic UI update before the Server Function completes, giving users instant feedback.
- **Revalidation**: After a Server Function mutates data, call `revalidatePath` or `revalidateTag` to refresh cached server-rendered content.
- **Error Boundaries**: Server Functions that throw errors can be caught by React Error Boundaries, allowing graceful error recovery.
- **Closure over server data**: Inline Server Functions in Server Components can close over server-side variables, binding data like user IDs without exposing them to the client.

## Real World Context

Consider a task management app where users can toggle tasks between complete and incomplete. When a user clicks the checkbox, you want the UI to update instantly (optimistic), send the mutation to the server (Server Function), and refresh the task list if other data changed (revalidation). This three-step pattern — optimistic update, server mutation, revalidation — is the bread and butter of modern React applications.

## Deep Dive

Combining Server Functions with optimistic updates:

```tsx
"use client";

import { useOptimistic } from "react";
import { toggleTask } from "./actions";

function TaskItem({ task }: { task: Task }) {
  const [optimisticDone, setOptimisticDone] = useOptimistic(
    task.done,
    (current: boolean, newValue: boolean) => newValue
  );

  return (
    <form action={async () => {
      setOptimisticDone(!optimisticDone);
      await toggleTask(task.id, !task.done);
    }}>
      <button type="submit">
        {optimisticDone ? "Done" : "Not Done"}: {task.title}
      </button>
    </form>
  );
}
```

Closures in Server Components bind server data securely:

```tsx
// Page.tsx (Server Component)
import { db } from "@/lib/db";
import { auth } from "@/lib/auth";

export default async function Page() {
  const userId = await auth.getUserId();

  async function addFavorite(formData: FormData) {
    "use server";
    const itemId = formData.get("itemId") as string;
    // userId is securely bound — the client never sees it
    await db.favorite.create({ data: { userId, itemId } });
  }

  const items = await db.items.findMany();
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          {item.name}
          <form action={addFavorite}>
            <input type="hidden" name="itemId" value={item.id} />
            <button type="submit">Favorite</button>
          </form>
        </li>
      ))}
    </ul>
  );
}
```

Handling errors gracefully:

```tsx
"use server";

export async function updateSettings(prevState: any, formData: FormData) {
  try {
    const theme = formData.get("theme") as string;
    await db.settings.update({ where: { userId: auth.userId }, data: { theme } });
    return { success: true, error: null };
  } catch (err) {
    return { success: false, error: "Failed to update settings. Please try again." };
  }
}
```

## Common Pitfalls

1. **Not revalidating after mutations** — If you mutate data with a Server Function but forget to call `revalidatePath` or `revalidateTag`, the cached page will show stale data until the cache expires.
2. **Exposing sensitive data in closures carelessly** — While closures are encrypted by React and frameworks, be mindful of what variables you close over. Avoid closing over entire user objects when you only need an ID.

## Best Practices

1. **Return structured state from Server Functions** — Instead of throwing errors, return objects like `{ success: boolean; error: string | null }` so the UI can handle both outcomes gracefully without Error Boundaries.
2. **Combine with `useOptimistic` for instant feedback** — For any toggle, like, or quick update, pair the Server Function with `useOptimistic` to eliminate perceived latency.

## Summary

- Combine Server Functions with `useOptimistic` for instant UI feedback while the server processes mutations.
- Use `revalidatePath` or `revalidateTag` after mutations to refresh cached server-rendered content.
- Inline Server Functions in Server Components can securely close over server-side data like user IDs.

## Code Examples

**Optimistic UI update combined with a Server Function**

```tsx
// Optimistic toggle with Server Function
"use client";
import { useOptimistic } from "react";
import { toggleLike } from "./actions";

function LikeButton({ liked, postId }: { liked: boolean; postId: string }) {
  const [optimisticLiked, setOptimistic] = useOptimistic(
    liked,
    (_current: boolean, next: boolean) => next
  );

  return (
    <form action={async () => {
      setOptimistic(!optimisticLiked);
      await toggleLike(postId);
    }}>
      <button type="submit">{optimisticLiked ? "Unlike" : "Like"}</button>
    </form>
  );
}
```


## Resources

- [Updating Data](https://react.dev/reference/rsc/server-functions#server-functions-in-forms) — Using Server Functions in forms for data mutations

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*