---
source_course: "svelte-5-snippets"
source_lesson: "svelte-5-snippets-children-prop"
---

# Default Content with children

In Svelte 5, content passed between component tags becomes the `children` snippet prop.

## The Old Way (Slots)

```svelte
<!-- Card.svelte (Svelte 4) -->
<div class="card">
  <slot />  <!-- Default slot -->
</div>

<!-- Usage -->
<Card>
  <p>This is the content</p>
</Card>
```

## The New Way (children)

```svelte
<!-- Card.svelte (Svelte 5) -->
<script>
  let { children } = $props();
</script>

<div class="card">
  {@render children()}
</div>
```

```svelte
<!-- Usage (same as before!) -->
<Card>
  <p>This is the content</p>
</Card>
```

The content between `<Card>` and `</Card>` automatically becomes the `children` prop.

## Optional children

Not all components have children. Handle this gracefully:

```svelte
<script>
  let { children } = $props();
</script>

<div class="container">
  {#if children}
    {@render children()}
  {:else}
    <p class="placeholder">No content provided</p>
  {/if}
</div>
```

Or more concisely:

```svelte
{@render children?.()}
```

## children with Other Props

```svelte
<!-- Button.svelte -->
<script>
  let { 
    children,
    variant = 'primary',
    disabled = false,
    onclick
  } = $props();
</script>

<button 
  class="btn btn-{variant}" 
  {disabled}
  {onclick}
>
  {@render children()}
</button>
```

```svelte
<!-- Usage -->
<Button variant="danger" onclick={handleDelete}>
  <TrashIcon /> Delete Item
</Button>
```

## Real-World Example: Modal

```svelte
<!-- Modal.svelte -->
<script>
  let { children, open = $bindable(false), title } = $props();
</script>

{#if open}
  <div class="modal-backdrop" onclick={() => open = false}>
    <div class="modal" onclick={(e) => e.stopPropagation()}>
      <header>
        <h2>{title}</h2>
        <button onclick={() => open = false}>×</button>
      </header>
      <main>
        {@render children()}
      </main>
    </div>
  </div>
{/if}
```

```svelte
<!-- Usage -->
<script>
  let showModal = $state(false);
</script>

<button onclick={() => showModal = true}>Open Modal</button>

<Modal bind:open={showModal} title="Confirm Action">
  <p>Are you sure you want to proceed?</p>
  <button onclick={() => showModal = false}>Cancel</button>
  <button onclick={confirm}>Confirm</button>
</Modal>
```

---

> 📘 *This lesson is part of the [Component Composition with Snippets](https://stanza.dev/courses/svelte-5-snippets) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*