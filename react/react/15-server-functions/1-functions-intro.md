---
source_course: "react"
source_lesson: "react-server-functions-intro"
---

# Server Functions

Define async functions that run on the server but can be called from client components. Perfect for form submissions, data mutations, and any server-side logic.

## Defining Server Functions

Use the `'use server'` directive:

```tsx
// In a separate file
'use server';

export async function createPost(formData: FormData) {
  // This runs on the server
}
```

Or inline in a Server Component:

```tsx
async function saveData() {
  'use server';
  // Server code here
}
```

## How They Work

1. React serializes the function reference
2. Client calls the reference
3. Request sent to server
4. Server executes the function
5. Result sent back to client

## Code Examples

**Server Function with useActionState**

```tsx
// actions.ts
'use server';

import { db } from '@/lib/db';
import { revalidatePath } from 'next/cache';

export async function createComment(
  prevState: { error?: string } | null,
  formData: FormData
) {
  const postId = formData.get('postId') as string;
  const content = formData.get('content') as string;
  
  // Validation
  if (!content || content.length < 3) {
    return { error: 'Comment must be at least 3 characters' };
  }
  
  // Database operation
  await db.comment.create({
    data: { postId, content }
  });
  
  // Revalidate the page to show new comment
  revalidatePath(`/posts/${postId}`);
  
  return { error: null };
}

// CommentForm.tsx
'use client';

import { useActionState } from 'react';
import { createComment } from './actions';

export function CommentForm({ postId }: { postId: string }) {
  const [state, formAction, isPending] = useActionState(
    createComment,
    null
  );
  
  return (
    <form action={formAction}>
      <input type="hidden" name="postId" value={postId} />
      <textarea 
        name="content" 
        placeholder="Write a comment..."
        disabled={isPending}
      />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Posting...' : 'Post Comment'}
      </button>
      {state?.error && <p className="error">{state.error}</p>}
    </form>
  );
}
```


## Resources

- [Server Functions](https://react.dev/reference/rsc/server-functions) — Official Server Functions guide

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*