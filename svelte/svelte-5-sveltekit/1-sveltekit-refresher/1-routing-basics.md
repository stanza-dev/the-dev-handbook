---
source_course: "svelte-5-sveltekit"
source_lesson: "svelte-5-sveltekit-routing-basics"
---

# SvelteKit's File-Based Router

SvelteKit uses your file system to define routes. The `src/routes` folder is your website's URL structure.

## How Files Map to URLs

```
src/routes/
├── +page.svelte          → /
├── about/
│   └── +page.svelte      → /about
├── blog/
│   ├── +page.svelte      → /blog
│   └── [slug]/
│       └── +page.svelte  → /blog/any-slug
└── api/
    └── users/
        └── +server.js    → /api/users (API endpoint)
```

## Special Files

| File | Purpose |
|------|--------|
| `+page.svelte` | The UI for a route |
| `+page.js` | Load data (runs on server AND client) |
| `+page.server.js` | Load data (server only) + form actions |
| `+layout.svelte` | Shared UI wrapper |
| `+layout.js` | Load data for layout |
| `+error.svelte` | Error page |
| `+server.js` | API endpoint |

## Dynamic Routes with Parameters

Wrap folder names in brackets for dynamic segments:

```
src/routes/
├── users/
│   └── [id]/
│       └── +page.svelte  → /users/123, /users/456, etc.
```

```svelte
<!-- src/routes/users/[id]/+page.svelte -->
<script>
  let { data } = $props();
</script>

<h1>User: {data.user.name}</h1>
```

```javascript
// src/routes/users/[id]/+page.server.js
export async function load({ params }) {
  const user = await getUser(params.id); // params.id is '123'
  return { user };
}
```

## Rest Parameters

Capture multiple path segments:

```
src/routes/docs/[...path]/+page.svelte
→ /docs/getting-started
→ /docs/api/components/button
```

```javascript
export function load({ params }) {
  // params.path = 'api/components/button'
}
```

📖 [Routing documentation](https://svelte.dev/docs/kit/routing)

---

> 📘 *This lesson is part of the [SvelteKit 2 & Svelte 5: The Perfect Duo](https://stanza.dev/courses/svelte-5-sveltekit) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*