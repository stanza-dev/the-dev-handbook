---
source_course: "svelte-5-sveltekit"
source_lesson: "svelte-5-sveltekit-load-functions"
---

# Loading Data in SvelteKit

SvelteKit's `load` function is how you fetch data for your pages. It runs before your component renders.

## Server Load Function

```javascript
// src/routes/posts/+page.server.js
export async function load({ fetch, params, cookies }) {
  // Access to server-only features
  const posts = await db.posts.findMany(); // Direct DB access
  
  // Or use fetch for API calls
  const response = await fetch('/api/posts');
  const data = await response.json();
  
  // Return data to the page
  return {
    posts: data.posts,
    totalCount: data.total
  };
}
```

## Universal Load Function

Runs on server (initial load) AND client (navigation):

```javascript
// src/routes/posts/+page.js
export async function load({ fetch, params }) {
  // Use the provided fetch - works on both server and client
  const response = await fetch('/api/posts');
  return { posts: await response.json() };
}
```

## Using Load Data in Components

```svelte
<!-- src/routes/posts/+page.svelte -->
<script>
  let { data } = $props();
</script>

<h1>Posts ({data.totalCount})</h1>

{#each data.posts as post}
  <article>
    <h2>{post.title}</h2>
    <p>{post.excerpt}</p>
  </article>
{/each}
```

## Load Function Parameters

```javascript
export async function load({
  params,    // Route parameters (/posts/[id] → params.id)
  url,       // URL object with searchParams
  fetch,     // Enhanced fetch (relative URLs work, credentials included)
  cookies,   // Read/write cookies (server only)
  request,   // Original request (server only)
  locals,    // Data from hooks (server only)
  parent,    // Data from parent layout's load
  depends,   // Declare dependencies for invalidation
  untrack    // Opt out of dependency tracking
}) {
  // ...
}
```

## Error Handling

```javascript
import { error } from '@sveltejs/kit';

export async function load({ params }) {
  const post = await getPost(params.id);
  
  if (!post) {
    throw error(404, 'Post not found');
  }
  
  return { post };
}
```

📖 [Load documentation](https://svelte.dev/docs/kit/load)

---

> 📘 *This lesson is part of the [SvelteKit 2 & Svelte 5: The Perfect Duo](https://stanza.dev/courses/svelte-5-sveltekit) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*