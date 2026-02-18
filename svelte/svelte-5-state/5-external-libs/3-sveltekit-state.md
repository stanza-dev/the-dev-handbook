---
source_course: "svelte-5-state"
source_lesson: "svelte-5-state-sveltekit-state"
---

# State Management Patterns in SvelteKit

SvelteKit has its own patterns for handling state, especially with load functions and form actions.

## Load Function Data

SvelteKit's `load` functions provide data to pages:

```javascript
// +page.js or +page.server.js
export async function load({ fetch }) {
  const response = await fetch('/api/products');
  const products = await response.json();
  
  return { products };
}
```

```svelte
<!-- +page.svelte -->
<script>
  let { data } = $props();
</script>

{#each data.products as product}
  <ProductCard {product} />
{/each}
```

## Form Actions for Mutations

```javascript
// +page.server.js
export const actions = {
  addToCart: async ({ request, cookies }) => {
    const formData = await request.formData();
    const productId = formData.get('productId');
    // Add to cart logic...
    return { success: true };
  }
};
```

```svelte
<!-- +page.svelte -->
<script>
  import { enhance } from '$app/forms';
  let { data, form } = $props();
</script>

<form method="POST" action="?/addToCart" use:enhance>
  <input type="hidden" name="productId" value={product.id} />
  <button type="submit">Add to Cart</button>
</form>

{#if form?.success}
  <p>Added to cart!</p>
{/if}
```

## Combining with Context

For client-side state that needs to persist across navigation:

```svelte
<!-- +layout.svelte -->
<script>
  import { setContext } from 'svelte';
  
  // Create cart state
  const cart = $state({
    items: [],
    add(product) {
      this.items = [...this.items, product];
    }
  });
  
  setContext('cart', cart);
</script>

<slot />
```

## $page Store for Navigation State

```svelte
<script>
  import { page } from '$app/stores';
</script>

<p>Current URL: {$page.url.pathname}</p>
<p>Route params: {JSON.stringify($page.params)}</p>
<p>Query string: {$page.url.searchParams.get('q')}</p>
```

## When to Use What in SvelteKit

| Data Type | Solution |
|-----------|----------|
| Server data (initial load) | `load` functions |
| Server mutations | Form actions |
| Client UI state | Local `$state` |
| Shared client state | Context |
| URL state | `$page.url` / `goto()` |
| Server cache | TanStack Query |

📖 [SvelteKit data loading](https://svelte.dev/docs/kit/load)

---

> 📘 *This lesson is part of the [Scalable State Management](https://stanza.dev/courses/svelte-5-state) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*