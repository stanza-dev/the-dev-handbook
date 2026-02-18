---
source_course: "svelte-5-sveltekit"
source_lesson: "svelte-5-sveltekit-api-routes"
---

# Building APIs with SvelteKit

Create REST API endpoints using `+server.js` files.

## Basic API Route

```javascript
// src/routes/api/posts/+server.js
import { json } from '@sveltejs/kit';

export async function GET() {
  const posts = await db.posts.findMany();
  return json(posts);
}

export async function POST({ request }) {
  const data = await request.json();
  const post = await db.posts.create({ data });
  return json(post, { status: 201 });
}
```

## HTTP Methods

```javascript
// +server.js
import { json, error } from '@sveltejs/kit';

export async function GET({ url }) {
  const limit = Number(url.searchParams.get('limit') ?? 10);
  const posts = await getPosts(limit);
  return json(posts);
}

export async function POST({ request }) {
  const data = await request.json();
  const post = await createPost(data);
  return json(post, { status: 201 });
}

export async function PUT({ request, params }) {
  const data = await request.json();
  const post = await updatePost(params.id, data);
  return json(post);
}

export async function DELETE({ params }) {
  await deletePost(params.id);
  return new Response(null, { status: 204 });
}
```

## Dynamic API Routes

```javascript
// src/routes/api/posts/[id]/+server.js
import { json, error } from '@sveltejs/kit';

export async function GET({ params }) {
  const post = await getPost(params.id);
  
  if (!post) {
    throw error(404, 'Post not found');
  }
  
  return json(post);
}
```

## Setting Headers

```javascript
export async function GET() {
  const data = await getData();
  
  return json(data, {
    headers: {
      'Cache-Control': 'max-age=3600',
      'X-Custom-Header': 'value'
    }
  });
}
```

## Handling Different Content Types

```javascript
export async function GET({ request }) {
  const accept = request.headers.get('Accept');
  const data = await getData();
  
  if (accept?.includes('text/csv')) {
    const csv = convertToCSV(data);
    return new Response(csv, {
      headers: { 'Content-Type': 'text/csv' }
    });
  }
  
  return json(data);
}
```

📖 [API routes documentation](https://svelte.dev/docs/kit/routing#server)

---

> 📘 *This lesson is part of the [SvelteKit 2 & Svelte 5: The Perfect Duo](https://stanza.dev/courses/svelte-5-sveltekit) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*