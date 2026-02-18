---
source_course: "svelte-5-snippets"
source_lesson: "svelte-5-snippets-global-styles"
---

# Breaking Out of Scoping

Sometimes you need styles to reach outside the component. Use `:global()`.

## Basic :global

```svelte
<style>
  /* Only affects elements in THIS component */
  p { color: blue; }
  
  /* Affects ALL matching elements */
  :global(p) { color: blue; }
</style>
```

## Targeting Nested Content

```svelte
<script>
  let { children } = $props();
</script>

<div class="wrapper">
  {@render children?.()}
</div>

<style>
  .wrapper {
    padding: 1rem;
  }
  
  /* Style any <p> inside .wrapper, even from snippets */
  .wrapper :global(p) {
    margin: 0.5rem 0;
  }
  
  /* Style first child of rendered content */
  .wrapper :global(> *:first-child) {
    margin-top: 0;
  }
</style>
```

## Common Patterns

### Prose Styling

```svelte
<!-- Prose.svelte -->
<script>
  let { children } = $props();
</script>

<article class="prose">
  {@render children?.()}
</article>

<style>
  .prose :global(h1) { font-size: 2rem; }
  .prose :global(h2) { font-size: 1.5rem; }
  .prose :global(p) { line-height: 1.6; }
  .prose :global(code) { background: #f0f0f0; }
  .prose :global(pre) { padding: 1rem; overflow-x: auto; }
  .prose :global(ul, ol) { padding-left: 2rem; }
</style>
```

### Third-Party Content

```svelte
<!-- MarkdownRenderer.svelte -->
<script>
  let { html } = $props();
</script>

<div class="markdown">
  {@html html}
</div>

<style>
  .markdown :global(table) {
    width: 100%;
    border-collapse: collapse;
  }
  .markdown :global(th, td) {
    border: 1px solid #ddd;
    padding: 8px;
  }
</style>
```

## When to Avoid :global

- Don't use for component-internal styling
- Avoid broad selectors like `:global(div)`
- Prefer component props for customization
- Use CSS custom properties for theming instead

```svelte
<!-- Better approach: CSS custom properties -->
<div class="card" style="--card-bg: {bgColor};">
  {@render children?.()}
</div>

<style>
  .card {
    background: var(--card-bg, white);
  }
</style>
```

---

> 📘 *This lesson is part of the [Component Composition with Snippets](https://stanza.dev/courses/svelte-5-snippets) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*