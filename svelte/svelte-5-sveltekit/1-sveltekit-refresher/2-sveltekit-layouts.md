---
source_course: "svelte-5-sveltekit"
source_lesson: "svelte-5-sveltekit-layouts"
---

# Layouts: Shared UI Structure

Layouts wrap pages and persist across navigation. Perfect for headers, sidebars, and footers.

## Basic Layout

```svelte
<!-- src/routes/+layout.svelte -->
<script>
  let { children } = $props();
</script>

<header>
  <nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
  </nav>
</header>

<main>
  {@render children()}
</main>

<footer>
  <p>© 2024 My App</p>
</footer>
```

## Svelte 5 Change: children Instead of slot

In Svelte 5, layouts receive `children` as a snippet prop:

```svelte
<!-- OLD Svelte 4 -->
<slot />

<!-- NEW Svelte 5 -->
{@render children()}
```

## Nested Layouts

Layouts nest automatically:

```
src/routes/
├── +layout.svelte         ← Root layout (applies to ALL pages)
├── +page.svelte           ← Home page
└── dashboard/
    ├── +layout.svelte     ← Dashboard layout (adds sidebar)
    ├── +page.svelte       ← /dashboard
    └── settings/
        └── +page.svelte   ← /settings (has BOTH layouts)
```

## Layout Groups

Group routes that share a layout without affecting the URL:

```
src/routes/
├── (marketing)/          ← Group - doesn't appear in URL
│   ├── +layout.svelte    ← Marketing layout
│   ├── +page.svelte      ← /
│   └── pricing/
│       └── +page.svelte  ← /pricing
├── (app)/                 ← Another group
│   ├── +layout.svelte    ← App layout (with sidebar)
│   └── dashboard/
│       └── +page.svelte  ← /dashboard
```

## Breaking Out of Layouts

Use `+page@.svelte` to reset to a specific layout:

```
src/routes/
├── +layout.svelte         ← Layout A
├── (group)/
│   ├── +layout.svelte     ← Layout B
│   └── page/
│       ├── +page.svelte   ← Uses A + B
│       └── +page@.svelte  ← Uses ONLY A (resets to root)
```

📖 [Layouts documentation](https://svelte.dev/docs/kit/routing#layout)

---

> 📘 *This lesson is part of the [SvelteKit 2 & Svelte 5: The Perfect Duo](https://stanza.dev/courses/svelte-5-sveltekit) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*