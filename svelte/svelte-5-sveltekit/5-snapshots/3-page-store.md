---
source_course: "svelte-5-sveltekit"
source_lesson: "svelte-5-sveltekit-page-store"
---

# Accessing Route Information with $page

The `$page` store gives you information about the current route.

## What's in $page?

```svelte
<script>
  import { page } from '$app/stores';
</script>

<!-- Current URL -->
<p>Path: {$page.url.pathname}</p>
<p>Search: {$page.url.searchParams.get('q')}</p>

<!-- Route parameters -->
<p>ID: {$page.params.id}</p>

<!-- Data from load functions -->
<p>User: {$page.data.user?.name}</p>

<!-- Form action result -->
<p>Form: {$page.form?.message}</p>

<!-- Error info (on error pages) -->
<p>Error: {$page.error?.message}</p>

<!-- Current route info -->
<p>Route: {$page.route.id}</p>
```

## Reactive Navigation UI

```svelte
<script>
  import { page } from '$app/stores';
  
  const navItems = [
    { href: '/', label: 'Home' },
    { href: '/about', label: 'About' },
    { href: '/blog', label: 'Blog' }
  ];
</script>

<nav>
  {#each navItems as item}
    <a 
      href={item.href} 
      class:active={$page.url.pathname === item.href}
    >
      {item.label}
    </a>
  {/each}
</nav>

<style>
  .active {
    font-weight: bold;
    color: var(--accent);
  }
</style>
```

## Accessing in Load Functions

```javascript
// Can't use $page store in load functions
// Instead, use the event object:
export async function load({ url, params, route }) {
  console.log(url.pathname);  // Current path
  console.log(params.id);     // Route params
  console.log(route.id);      // Route ID like '/posts/[id]'
}
```

## $page vs data Props

```svelte
<script>
  import { page } from '$app/stores';
  
  // Both work, but...
  let { data } = $props();   // Recommended: direct access
  // $page.data             // Works but less direct
</script>
```

Use `$props()` for data in page components. Use `$page` for:
- Navigation state outside page components
- Route info (pathname, params)
- Reactive nav highlighting

📖 [$page documentation](https://svelte.dev/docs/kit/$app-stores#page)

---

> 📘 *This lesson is part of the [SvelteKit 2 & Svelte 5: The Perfect Duo](https://stanza.dev/courses/svelte-5-sveltekit) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*