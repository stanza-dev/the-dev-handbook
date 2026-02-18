---
source_course: "svelte-5-sveltekit"
source_lesson: "svelte-5-sveltekit-rendering-modes"
---

# Understanding Rendering in SvelteKit

SvelteKit supports multiple rendering strategies. Understanding them is crucial for performance and SEO.

## Server-Side Rendering (SSR) - Default

Pages are rendered on the server, sent as HTML, then "hydrated" on the client:

```
1. Browser requests /about
2. Server runs +page.server.js load function
3. Server renders +page.svelte to HTML
4. Server sends complete HTML to browser
5. Browser displays HTML immediately (fast!)
6. JavaScript loads and "hydrates" (makes interactive)
```

**Benefits:**
- Fast initial paint (user sees content immediately)
- SEO-friendly (search engines see content)
- Works without JavaScript

## Client-Side Rendering (CSR)

Disable SSR for specific pages:

```javascript
// +page.js
export const ssr = false;
```

Now the page is rendered entirely in the browser. Useful for:
- Dashboard pages (no SEO needed)
- Pages using browser-only APIs
- Reducing server load

## Prerendering (Static Generation)

Generate HTML at build time:

```javascript
// +page.js
export const prerender = true;
```

The page becomes a static HTML file. Perfect for:
- Marketing pages
- Blog posts
- Documentation
- Any content that doesn't change per-request

## Combining Strategies

```javascript
// +page.js
export const prerender = true;  // Generate at build time
export const ssr = true;        // Default (can omit)
```

```javascript
// +layout.js - Applies to all child pages
export const ssr = false;
export const prerender = false;
```

## Checking the Environment

```svelte
<script>
  import { browser } from '$app/environment';
  
  if (browser) {
    // Runs only in the browser
    console.log('Window:', window.innerWidth);
  }
</script>
```

📖 [Page options documentation](https://svelte.dev/docs/kit/page-options)

---

> 📘 *This lesson is part of the [SvelteKit 2 & Svelte 5: The Perfect Duo](https://stanza.dev/courses/svelte-5-sveltekit) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*