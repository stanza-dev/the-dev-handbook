---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-layouts"
---

# Wrapping Pages with Layouts

Most apps have shared UI elements — navigation, footers, sidebars. Instead of duplicating them on every page, use **layouts**.

## Creating a Layout

Add a `+layout.svelte` file next to your pages:

```svelte
<!-- src/routes/+layout.svelte -->
<script>
  let { children } = $props();
</script>

<nav>
  <a href="/">Home</a>
  <a href="/about">About</a>
  <a href="/contact">Contact</a>
</nav>

<main>
  {@render children()}
</main>

<footer>
  <p>© 2024 My App</p>
</footer>
```

**Key concept**: `{@render children()}` renders the current page content!

## Layout Inheritance

Layouts apply to ALL pages in their directory and subdirectories:

```
src/routes/
├── +layout.svelte    ← Applies to everything
├── +page.svelte      ← Wrapped by layout
├── about/
│   └── +page.svelte  ← Also wrapped
└── blog/
    ├── +layout.svelte  ← Nested layout (adds to parent)
    ├── +page.svelte
    └── [slug]/
        └── +page.svelte
```

## Nested Layouts

```svelte
<!-- src/routes/blog/+layout.svelte -->
<script>
  let { children } = $props();
</script>

<div class="blog-container">
  <aside>
    <h3>Categories</h3>
    <!-- Sidebar content -->
  </aside>
  
  <article>
    {@render children()}
  </article>
</div>
```

Pages in `/blog/*` will have BOTH the root layout AND the blog layout.

## Layout Data

Just like pages, layouts can have load functions:

```js
// src/routes/+layout.server.js
export async function load() {
  return {
    user: await getUser()  // Available in all child pages!
  };
}
```

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*