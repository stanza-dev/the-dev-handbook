---
source_course: "svelte-5-state"
source_lesson: "svelte-5-state-lifting-state"
---

# Lifting State Up

The most common pattern for sharing state is lifting it to a common ancestor. This is where you should start before reaching for more complex solutions.

## The Problem: Sibling Communication

Two sibling components need to share data:

```svelte
<!-- ProductList.svelte -->
<script>
  let selectedProductId = $state(null); // How does ProductDetails know?
</script>
```

```svelte
<!-- ProductDetails.svelte -->
<script>
  // I need to know which product is selected!
</script>
```

## The Solution: Lift to Parent

```svelte
<!-- Parent.svelte -->
<script>
  import ProductList from './ProductList.svelte';
  import ProductDetails from './ProductDetails.svelte';
  
  // State lives here - accessible to both children
  let selectedProductId = $state(null);
</script>

<ProductList 
  {selectedProductId}
  onselect={(id) => selectedProductId = id}
/>

<ProductDetails productId={selectedProductId} />
```

```svelte
<!-- ProductList.svelte -->
<script>
  let { selectedProductId, onselect } = $props();
</script>

{#each products as product}
  <div 
    class:selected={product.id === selectedProductId}
    onclick={() => onselect(product.id)}
  >
    {product.name}
  </div>
{/each}
```

```svelte
<!-- ProductDetails.svelte -->
<script>
  let { productId } = $props();
  let product = $derived(products.find(p => p.id === productId));
</script>

{#if product}
  <h2>{product.name}</h2>
  <p>{product.description}</p>
{/if}
```

## Data Flow Visualization

```
       ┌─────────────────────────┐
       │        Parent           │
       │  selectedProductId = 5  │
       └───────────┬─────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
        ▼                     ▼
┌───────────────┐     ┌───────────────┐
│  ProductList  │     │ProductDetails │
│ (reads & sets)│     │ (reads only)  │
└───────────────┘     └───────────────┘
```

## Benefits

1. **Single source of truth** - State exists in one place
2. **Predictable data flow** - Always top-down
3. **Easy to debug** - You know exactly where state lives
4. **No hidden dependencies** - Everything is explicit

📖 [Props documentation](https://svelte.dev/docs/svelte/$props)

---

> 📘 *This lesson is part of the [Scalable State Management](https://stanza.dev/courses/svelte-5-state) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*