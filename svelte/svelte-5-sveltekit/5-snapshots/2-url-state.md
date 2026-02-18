---
source_course: "svelte-5-sveltekit"
source_lesson: "svelte-5-sveltekit-url-state"
---

# URL State: Shareable & Bookmarkable

For state that should be shareable or bookmarkable, put it in the URL.

## Reading URL State

```javascript
// +page.server.js
export async function load({ url }) {
  const page = Number(url.searchParams.get('page') ?? 1);
  const search = url.searchParams.get('q') ?? '';
  const sort = url.searchParams.get('sort') ?? 'date';
  
  const products = await getProducts({ page, search, sort });
  
  return { products, page, search, sort };
}
```

```svelte
<!-- +page.svelte -->
<script>
  let { data } = $props();
</script>

<p>Showing results for "{data.search}"</p>
```

## Updating URL State

```svelte
<script>
  import { goto } from '$app/navigation';
  import { page } from '$app/stores';
  
  let { data } = $props();
  
  function updateSearch(newSearch) {
    const url = new URL($page.url);
    url.searchParams.set('q', newSearch);
    url.searchParams.set('page', '1'); // Reset to page 1
    goto(url);
  }
  
  function changePage(newPage) {
    const url = new URL($page.url);
    url.searchParams.set('page', String(newPage));
    goto(url);
  }
</script>

<input 
  value={data.search} 
  onchange={(e) => updateSearch(e.currentTarget.value)}
/>

<button onclick={() => changePage(data.page + 1)}>Next Page</button>
```

## Using Links Instead of goto()

For simple cases, regular links work:

```svelte
<script>
  let { data } = $props();
  import { page } from '$app/stores';
</script>

<nav class="pagination">
  {#each Array(data.totalPages) as _, i}
    <a 
      href="?page={i + 1}" 
      class:active={$page.url.searchParams.get('page') == i + 1}
    >
      {i + 1}
    </a>
  {/each}
</nav>
```

## goto() Options

```javascript
import { goto } from '$app/navigation';

// Replace history entry (no back button entry)
goto(url, { replaceState: true });

// Keep scroll position
goto(url, { noScroll: true });

// Prevent load function from re-running
goto(url, { invalidateAll: false });
```

📖 [Navigation documentation](https://svelte.dev/docs/kit/$app-navigation)

---

> 📘 *This lesson is part of the [SvelteKit 2 & Svelte 5: The Perfect Duo](https://stanza.dev/courses/svelte-5-sveltekit) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*