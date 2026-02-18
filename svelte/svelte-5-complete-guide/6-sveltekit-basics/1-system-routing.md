---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-file-system-routing"
---

# SvelteKit: The Full-Stack Framework

SvelteKit is the official framework for building Svelte applications. It provides:
- File-based routing
- Server-side rendering (SSR)
- Data loading
- Form handling
- And much more!

## How Routing Works

SvelteKit uses your **file system** to define routes. Files in `src/routes/` become pages in your app:

```
src/routes/
├── +page.svelte          → /
├── about/
│   └── +page.svelte      → /about
├── blog/
│   ├── +page.svelte      → /blog
│   └── [slug]/
│       └── +page.svelte  → /blog/hello-world, /blog/my-post, etc.
└── contact/
    └── +page.svelte      → /contact
```

## The +page.svelte File

Every route needs a `+page.svelte` file — this is the component that renders for that URL:

```svelte
<!-- src/routes/about/+page.svelte -->
<h1>About Us</h1>
<p>Welcome to our company!</p>
```

## Dynamic Routes

Square brackets create **dynamic segments** that match any value:

```
src/routes/blog/[slug]/+page.svelte
```

This matches:
- `/blog/hello-world` → slug = "hello-world"
- `/blog/my-first-post` → slug = "my-first-post"
- `/blog/123` → slug = "123"

Access the parameter in your page:

```svelte
<!-- src/routes/blog/[slug]/+page.svelte -->
<script>
  let { data } = $props();
</script>

<h1>{data.post.title}</h1>
```

## Nested Routes

Folders create URL segments:

```
src/routes/
└── dashboard/
    ├── +page.svelte           → /dashboard
    ├── settings/
    │   └── +page.svelte       → /dashboard/settings
    └── profile/
        └── +page.svelte       → /dashboard/profile
```

## Resources

- [SvelteKit Routing](https://svelte.dev/docs/kit/routing) — Official SvelteKit routing documentation.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*