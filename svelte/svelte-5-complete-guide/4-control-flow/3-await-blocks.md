---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-await-blocks"
---

# Async Data in Templates

Modern apps often fetch data from APIs. Svelte's `{#await}` block lets you handle loading, success, and error states directly in your template.

## Basic Await Block

```svelte
<script>
  async function fetchUser() {
    const res = await fetch('/api/user');
    return res.json();
  }

  let userPromise = fetchUser();
</script>

{#await userPromise}
  <p>Loading user...</p>
{:then user}
  <p>Hello, {user.name}!</p>
{:catch error}
  <p>Error: {error.message}</p>
{/await}
```

## The Three States

| State | Block | When it shows |
|-------|-------|---------------|
| Pending | `{#await promise}` | While waiting |
| Resolved | `{:then value}` | On success |
| Rejected | `{:catch error}` | On failure |

## Skip the Loading State

If you don't need a loading indicator:

```svelte
{#await userPromise then user}
  <p>Hello, {user.name}!</p>
{/await}
```

Nothing shows while loading; content appears once resolved.

## Real Example: Fetching Posts

```svelte
<script>
  let postsPromise = fetch('/api/posts')
    .then(r => r.json());
</script>

<h1>Blog Posts</h1>

{#await postsPromise}
  <div class="skeleton-list">
    <div class="skeleton-item"></div>
    <div class="skeleton-item"></div>
    <div class="skeleton-item"></div>
  </div>
{:then posts}
  {#each posts as post}
    <article>
      <h2>{post.title}</h2>
      <p>{post.excerpt}</p>
    </article>
  {/each}
{:catch error}
  <div class="error">
    <p>Failed to load posts.</p>
    <button onclick={() => postsPromise = fetch('/api/posts').then(r => r.json())}>
      Try Again
    </button>
  </div>
{/await}
```

## Refreshing Data

To refetch, assign a new promise:

```svelte
<script>
  let dataPromise = $state(fetchData());
  
  function refresh() {
    dataPromise = fetchData();  // New promise triggers re-render
  }
</script>

<button onclick={refresh}>Refresh</button>

{#await dataPromise}
  Loading...
{:then data}
  {data.value}
{/await}
```

## Resources

- [{#await} Documentation](https://svelte.dev/docs/svelte/await) — Official docs for handling async operations in templates.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*