---
source_course: "svelte-5-migration"
source_lesson: "svelte-5-migration-slots-migration"
---

# Slots to Snippets

Svelte 5 replaces `<slot>` with snippets for component composition.

## Default Slot

```svelte
<!-- Svelte 4: Component -->
<div class="card">
  <slot />
</div>

<!-- Svelte 4: Usage -->
<Card>Content here</Card>
```

```svelte
<!-- Svelte 5: Component -->
<script>
  let { children } = $props();
</script>

<div class="card">
  {@render children()}
</div>

<!-- Svelte 5: Usage -->
<Card>Content here</Card>
```

## Named Slots

```svelte
<!-- Svelte 4: Component -->
<header><slot name="header" /></header>
<main><slot /></main>
<footer><slot name="footer" /></footer>

<!-- Svelte 4: Usage -->
<Layout>
  <h1 slot="header">Title</h1>
  <p>Main content</p>
  <p slot="footer">Footer</p>
</Layout>
```

```svelte
<!-- Svelte 5: Component -->
<script>
  let { header, children, footer } = $props();
</script>

<header>{@render header?.()}</header>
<main>{@render children()}</main>
<footer>{@render footer?.()}</footer>

<!-- Svelte 5: Usage -->
<Layout>
  {#snippet header()}
    <h1>Title</h1>
  {/snippet}
  
  <p>Main content</p>
  
  {#snippet footer()}
    <p>Footer</p>
  {/snippet}
</Layout>
```

## Slot Props (Passing Data Back)

```svelte
<!-- Svelte 4 -->
<slot item={currentItem} index={i} />

<!-- Usage -->
<List let:item let:index>
  <div>{index}: {item.name}</div>
</List>
```

```svelte
<!-- Svelte 5: Component -->
<script>
  let { children } = $props();
</script>

{#each items as item, index}
  {@render children(item, index)}
{/each}

<!-- Svelte 5: Usage -->
<List>
  {#snippet children(item, index)}
    <div>{index}: {item.name}</div>
  {/snippet}
</List>
```

📖 [Snippets documentation](https://svelte.dev/docs/svelte/snippet)

---

> 📘 *This lesson is part of the [Moving to Svelte 5](https://stanza.dev/courses/svelte-5-migration) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*