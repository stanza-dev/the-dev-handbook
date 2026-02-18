---
source_course: "svelte-5-sveltekit"
source_lesson: "svelte-5-sveltekit-streaming"
---

# Streaming: Don't Block on Slow Data

Instead of waiting for all data before rendering, you can stream slow data while showing fast data immediately.

## The Problem: Slow Data Blocks Everything

```javascript
// ❌ This blocks until BOTH are ready
export async function load() {
  const posts = await getPostsFast();     // 100ms
  const analytics = await getAnalytics(); // 3000ms
  
  return { posts, analytics }; // User waits 3100ms!
}
```

## The Solution: Stream Slow Data

```javascript
// ✅ Return promise without awaiting
export async function load() {
  const posts = await getPostsFast(); // Wait for this (fast)
  
  return {
    posts,
    streamed: {
      analytics: getAnalytics() // Don't await! (slow)
    }
  };
}
```

## Handling Streamed Data in Components

```svelte
<script>
  let { data } = $props();
</script>

<!-- This renders immediately -->
{#each data.posts as post}
  <article>{post.title}</article>
{/each}

<!-- This streams in when ready -->
{#await data.streamed.analytics}
  <div class="skeleton">Loading analytics...</div>
{:then analytics}
  <div class="stats">
    <p>Views: {analytics.views}</p>
    <p>Visitors: {analytics.visitors}</p>
  </div>
{:catch error}
  <p class="error">Failed to load analytics</p>
{/await}
```

## Multiple Streamed Values

```javascript
export async function load() {
  return {
    posts: await getPosts(),
    streamed: {
      comments: getComments(),    // Stream independently
      analytics: getAnalytics(),
      recommendations: getRecommendations()
    }
  };
}
```

```svelte
<!-- Each stream resolves independently -->
{#await data.streamed.comments}...
{#await data.streamed.analytics}...
{#await data.streamed.recommendations}...
```

## When to Stream vs Wait

| Scenario | Strategy |
|----------|----------|
| Critical above-fold content | `await` (block) |
| Below-fold content | Stream |
| Personalization data | Stream |
| Analytics/metrics | Stream |
| Secondary features | Stream |

📖 [Streaming documentation](https://svelte.dev/docs/kit/load#Streaming-with-promises)

---

> 📘 *This lesson is part of the [SvelteKit 2 & Svelte 5: The Perfect Duo](https://stanza.dev/courses/svelte-5-sveltekit) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*