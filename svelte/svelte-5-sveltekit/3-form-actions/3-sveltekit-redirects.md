---
source_course: "svelte-5-sveltekit"
source_lesson: "svelte-5-sveltekit-redirects"
---

# Redirecting After Form Submission

Often you want to redirect users after a successful action.

## Using redirect()

```javascript
import { redirect, fail } from '@sveltejs/kit';

export const actions = {
  create: async ({ request }) => {
    const data = await request.formData();
    const title = data.get('title');
    
    if (!title) {
      return fail(400, { error: 'Title required' });
    }
    
    const post = await db.posts.create({ title });
    
    // Redirect to the new post
    throw redirect(303, `/posts/${post.id}`);
  }
};
```

## Status Codes for Redirects

- `303` - See Other (recommended for POST → GET)
- `307` - Temporary Redirect (keeps method)
- `308` - Permanent Redirect (keeps method)

## Conditional Redirects

```javascript
export const actions = {
  login: async ({ request, cookies, url }) => {
    // ... authenticate user ...
    
    // Redirect to where they came from, or home
    const redirectTo = url.searchParams.get('redirect') || '/';
    throw redirect(303, redirectTo);
  }
};
```

## Handling Redirects with use:enhance

```svelte
<script>
  import { enhance } from '$app/forms';
  import { goto } from '$app/navigation';
</script>

<form method="POST" use:enhance={() => {
  return async ({ result }) => {
    if (result.type === 'redirect') {
      // Custom redirect handling
      await goto(result.location, { replaceState: true });
    }
  };
}}>
```

## Redirects in Load Functions

```javascript
// +page.server.js
import { redirect } from '@sveltejs/kit';

export async function load({ locals }) {
  if (!locals.user) {
    // Not logged in, redirect to login
    throw redirect(303, '/login');
  }
  
  return { user: locals.user };
}
```

📖 [Redirect documentation](https://svelte.dev/docs/kit/load#Redirects)

---

> 📘 *This lesson is part of the [SvelteKit 2 & Svelte 5: The Perfect Duo](https://stanza.dev/courses/svelte-5-sveltekit) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*