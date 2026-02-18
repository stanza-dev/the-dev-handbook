---
source_course: "vue-ssr-nuxt"
source_lesson: "vue-ssr-nuxt-what-is-ssr"
---

# What is Server-Side Rendering?

Server-side rendering (SSR) generates HTML on the server instead of in the browser, improving initial load time and SEO.

## Client-Side Rendering (CSR) vs SSR

### CSR (Traditional SPA)

```
Browser requests page
      ↓
Server sends minimal HTML + JS bundle
      ↓
Browser downloads JS
      ↓
JS executes, fetches data
      ↓
JS renders content
      ↓
User sees content (slow!)
```

**Issues:**
- Blank page until JS loads
- Poor SEO (crawlers see empty HTML)
- Slow time-to-content

### SSR

```
Browser requests page
      ↓
Server runs Vue, fetches data
      ↓
Server generates full HTML
      ↓
Browser receives complete HTML
      ↓
User sees content (fast!)
      ↓
JS loads and "hydrates"
      ↓
Page becomes interactive
```

**Benefits:**
- Fast initial content
- SEO-friendly
- Better perceived performance

## Key Concepts

### Hydration

"Hydration" is when Vue takes over the server-rendered HTML:

1. Server sends HTML with content
2. Browser shows static HTML immediately
3. Vue JS bundle loads
4. Vue "hydrates" - attaches event listeners, makes it reactive
5. Page is now fully interactive

### Universal/Isomorphic Code

Code that runs on both server and client. Important considerations:

```javascript
// ❌ Won't work on server - no window
const width = window.innerWidth

// ✅ Check environment first
if (typeof window !== 'undefined') {
  const width = window.innerWidth
}

// ✅ Or use Nuxt's process.client
if (process.client) {
  const width = window.innerWidth
}
```

## Rendering Modes

| Mode | When Rendered | Use Case |
|------|---------------|----------|
| SSR | On each request | Dynamic content |
| SSG | At build time | Static content |
| ISR | On-demand + cache | Semi-dynamic |
| CSR | In browser | Dashboard, apps |

### SSR (Server-Side Rendering)

```
Request → Server renders → Fresh HTML
```

Good for: Dynamic content, personalized pages

### SSG (Static Site Generation)

```
Build time → Pre-render all pages → Serve static files
```

Good for: Blogs, documentation, marketing sites

### ISR (Incremental Static Regeneration)

```
First request → Render & cache → Serve cached until stale
```

Good for: Large sites with semi-dynamic content

### Hybrid Rendering

Nuxt 3 allows mixing modes per-route:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    '/': { prerender: true },           // SSG
    '/blog/**': { isr: 3600 },          // ISR (1 hour)
    '/dashboard/**': { ssr: false },    // CSR only
    '/api/**': { cors: true }           // API routes
  }
})
```

## When to Use SSR

✅ **Use SSR when:**
- SEO is important
- First contentful paint matters
- Content changes frequently
- Social media sharing (meta tags)

❌ **Skip SSR when:**
- Building internal tools/dashboards
- Content is behind authentication
- Real-time interactive apps
- Simple static sites (use SSG)

## Resources

- [Vue SSR Guide](https://vuejs.org/guide/scaling-up/ssr.html) — Official Vue SSR documentation

---

> 📘 *This lesson is part of the [Vue Server-Side Rendering & Nuxt](https://stanza.dev/courses/vue-ssr-nuxt) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*