---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-loading-data"
---

# Fetching Data Before Rendering

SvelteKit's `load` functions let you fetch data on the server before your page renders. This enables:
- SEO-friendly server-side rendering
- Faster initial page loads
- Secure data fetching (API keys stay on server)

## Creating a Load Function

Add a `+page.server.js` file alongside your `+page.svelte`:

```js
// src/routes/blog/+page.server.js
export async function load() {
  // This runs on the server!
  const posts = await db.posts.findMany();
  
  return {
    posts
  };
}
```

## Using Load Data in Your Page

The returned object is available via the `data` prop:

```svelte
<!-- src/routes/blog/+page.svelte -->
<script>
  let { data } = $props();
</script>

<h1>Blog Posts</h1>

{#each data.posts as post}
  <article>
    <h2>{post.title}</h2>
    <p>{post.excerpt}</p>
    <a href="/blog/{post.slug}">Read more</a>
  </article>
{/each}
```

## Accessing Route Parameters

The load function receives useful information:

```js
// src/routes/blog/[slug]/+page.server.js
export async function load({ params }) {
  const post = await db.posts.findOne({
    where: { slug: params.slug }  // Access dynamic segment
  });
  
  if (!post) {
    throw error(404, 'Post not found');
  }
  
  return { post };
}
```

## Universal vs Server Load

| File | Runs On | Use For |
|------|---------|--------|
| `+page.server.js` | Server only | Database queries, secrets |
| `+page.js` | Server + Browser | Public APIs, client navigation |

```js
// +page.js (runs on server AND browser)
export async function load({ fetch }) {
  const res = await fetch('/api/posts');  // Use SvelteKit's fetch
  const posts = await res.json();
  return { posts };
}
```

## Reactive Data

When using data in reactive expressions, use `$derived`:

```svelte
<script>
  let { data } = $props();
  
  // Recalculates when data.posts changes (e.g., on navigation)
  let postCount = $derived(data.posts.length);
</script>
```

## Resources

- [Loading Data](https://svelte.dev/docs/kit/load) — Complete guide to data loading in SvelteKit.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*