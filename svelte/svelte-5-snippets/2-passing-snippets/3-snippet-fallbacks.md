---
source_course: "svelte-5-snippets"
source_lesson: "svelte-5-snippets-snippet-fallbacks"
---

# Handling Missing Snippets

Good components provide sensible defaults when snippets aren't provided.

## Conditional Rendering

```svelte
<script>
  let { header, children, footer } = $props();
</script>

<article>
  {#if header}
    <header>{@render header()}</header>
  {/if}
  
  <main>
    {#if children}
      {@render children()}
    {:else}
      <p class="placeholder">No content provided</p>
    {/if}
  </main>
  
  {#if footer}
    <footer>{@render footer()}</footer>
  {/if}
</article>
```

## Optional Chaining

For quick optional rendering:

```svelte
{@render header?.()}
{@render children?.()}
{@render footer?.()}
```

## Default Snippet Props

Define default snippets in props:

```svelte
<script>
  // Default implementations
  const defaultEmpty = () => {
    // This doesn't work - can't define snippets in script
  };
  
  let { 
    items,
    renderItem,  // Required
    empty        // Optional
  } = $props();
</script>

{#each items as item}
  {@render renderItem(item)}
{:else}
  {#if empty}
    {@render empty()}
  {:else}
    <p>No items</p>  <!-- Inline fallback -->
  {/if}
{/each}
```

## Component with Rich Defaults

```svelte
<!-- Alert.svelte -->
<script>
  let { 
    type = 'info',
    title,
    icon,
    children,
    action
  } = $props();
  
  const defaultIcons = {
    info: 'ℹ️',
    success: '✅',
    warning: '⚠️',
    error: '❌'
  };
</script>

<div class="alert alert-{type}">
  <span class="icon">
    {#if icon}
      {@render icon()}
    {:else}
      {defaultIcons[type]}
    {/if}
  </span>
  
  <div class="content">
    {#if title}
      <strong>{title}</strong>
    {/if}
    {@render children?.()}
  </div>
  
  {#if action}
    <div class="action">
      {@render action()}
    </div>
  {/if}
</div>
```

```svelte
<!-- Simple usage -->
<Alert type="success" title="Saved!">
  Your changes have been saved.
</Alert>

<!-- With custom icon and action -->
<Alert type="warning">
  {#snippet icon()}
    <CustomWarningIcon />
  {/snippet}
  
  Your session is about to expire.
  
  {#snippet action()}
    <button onclick={extendSession}>Extend Session</button>
  {/snippet}
</Alert>
```

---

> 📘 *This lesson is part of the [Component Composition with Snippets](https://stanza.dev/courses/svelte-5-snippets) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*