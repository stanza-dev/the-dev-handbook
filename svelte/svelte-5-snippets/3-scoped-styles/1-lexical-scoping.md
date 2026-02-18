---
source_course: "svelte-5-snippets"
source_lesson: "svelte-5-snippets-lexical-scoping"
---

# Where Do Snippet Styles Come From?

Snippets have **lexical scoping** — they use the styles of the file where they're DEFINED, not where they're RENDERED.

## The Key Insight

```svelte
<!-- App.svelte -->
<script>
  import Card from './Card.svelte';
</script>

<Card>
  {#snippet header()}
    <h2 class="title">My Title</h2>  <!-- Styled by App.svelte -->
  {/snippet}
  
  <p class="content">Body text</p>  <!-- Styled by App.svelte -->
</Card>

<style>
  .title { color: blue; }    /* Applied to header snippet */
  .content { font-size: 14px; } /* Applied to children */
</style>
```

```svelte
<!-- Card.svelte -->
<script>
  let { header, children } = $props();
</script>

<div class="card">
  {@render header?.()}
  {@render children?.()}
</div>

<style>
  .card { border: 1px solid gray; }  /* Only affects .card */
  .title { color: red; }  /* NOT applied to the snippet's h2! */
</style>
```

The `.title` in `App.svelte` is applied, not the one in `Card.svelte`.

## Why This Matters

Lexical scoping is actually a **feature**:

1. **Predictability**: The snippet author controls its styling
2. **No leaking**: Component styles can't accidentally affect passed content
3. **Intentional design**: If Card wants to style headers, it should provide an API

## Comparing to Slots

This is a major improvement over Svelte 4 slots, where styling was confusing:

```svelte
<!-- Svelte 4: Slot content styling was unpredictable -->
<Card>
  <h2 slot="header">Title</h2>  <!-- Whose styles apply? -->
</Card>
```

## When You Need Component Styles

If the component SHOULD style the passed content, use wrapper elements:

```svelte
<!-- Card.svelte -->
<script>
  let { header, children } = $props();
</script>

<div class="card">
  <div class="card-header">  <!-- Component controls wrapper -->
    {@render header?.()}
  </div>
  <div class="card-body">
    {@render children?.()}
  </div>
</div>

<style>
  .card-header {
    padding: 1rem;
    background: #f5f5f5;
    border-bottom: 1px solid #ddd;
  }
</style>
```

---

> 📘 *This lesson is part of the [Component Composition with Snippets](https://stanza.dev/courses/svelte-5-snippets) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*