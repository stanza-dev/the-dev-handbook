---
source_course: "svelte-5-performance"
source_lesson: "svelte-5-performance-lazy-loading"
---

# Code Splitting with Lazy Loading

Don't load everything upfront. Load components only when needed.

## Dynamic Imports

```svelte
<script>
  let showChart = $state(false);
  let ChartComponent = $state(null);
  
  async function loadChart() {
    showChart = true;
    // Only loads when needed!
    const module = await import('./HeavyChart.svelte');
    ChartComponent = module.default;
  }
</script>

<button onclick={loadChart}>Show Chart</button>

{#if showChart}
  {#if ChartComponent}
    <svelte:component this={ChartComponent} />
  {:else}
    <p>Loading chart...</p>
  {/if}
{/if}
```

## SvelteKit Automatic Code Splitting

SvelteKit automatically code-splits by route:

```
src/routes/
├── +page.svelte          ← Loaded immediately
├── dashboard/
│   └── +page.svelte      ← Loaded when user visits /dashboard
├── admin/
│   └── +page.svelte      ← Loaded when user visits /admin
```

## Prefetching

SvelteKit prefetches links on hover:

```svelte
<!-- Prefetched when user hovers -->
<a href="/dashboard">Dashboard</a>

<!-- Disable prefetching -->
<a href="/dashboard" data-sveltekit-preload-data="off">Dashboard</a>

<!-- Prefetch on viewport enter -->
<a href="/dashboard" data-sveltekit-preload-data="hover">Dashboard</a>
```

## Heavy Component Patterns

```svelte
<script>
  let isVisible = $state(false);
  
  // Intersection Observer for viewport-based loading
  function watchVisibility(node) {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) {
          isVisible = true;
          observer.disconnect();
        }
      },
      { threshold: 0.1 }
    );
    
    observer.observe(node);
    return { destroy: () => observer.disconnect() };
  }
</script>

<div use:watchVisibility>
  {#if isVisible}
    {#await import('./HeavyComponent.svelte') then { default: Heavy }}
      <Heavy />
    {/await}
  {:else}
    <div class="placeholder">Loading...</div>
  {/if}
</div>
```

📖 [Code splitting](https://svelte.dev/docs/kit/load#Streaming-with-promises)

---

> 📘 *This lesson is part of the [Under the Hood & Performance](https://stanza.dev/courses/svelte-5-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*