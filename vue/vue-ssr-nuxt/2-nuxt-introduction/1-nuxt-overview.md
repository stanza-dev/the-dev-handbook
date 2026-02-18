---
source_course: "vue-ssr-nuxt"
source_lesson: "vue-ssr-nuxt-nuxt-overview"
---

# What is Nuxt?

Nuxt is a framework built on Vue that provides SSR, SSG, file-based routing, and many developer experience improvements out of the box.

## Why Nuxt?

### Without Nuxt (Manual SSR)

- Configure webpack/Vite for SSR
- Set up Node.js server
- Handle hydration manually
- Configure routing
- Manage head/meta tags
- Handle data fetching
- Set up development environment

### With Nuxt

- ✅ SSR/SSG/ISR configured
- ✅ File-based routing
- ✅ Auto-imports
- ✅ Built-in head management
- ✅ Data fetching composables
- ✅ API routes
- ✅ TypeScript support
- ✅ Development tools

## Creating a Nuxt Project

```bash
npx nuxi@latest init my-nuxt-app
cd my-nuxt-app
npm install
npm run dev
```

## Project Structure

```
my-nuxt-app/
├── .nuxt/              # Build output (gitignored)
├── assets/             # Uncompiled assets (Sass, images)
├── components/         # Auto-imported Vue components
├── composables/        # Auto-imported composables
├── layouts/            # Layout components
├── middleware/         # Route middleware
├── pages/              # File-based routes
├── plugins/            # Vue plugins
├── public/             # Static files
├── server/             # Server routes & middleware
│   ├── api/            # API endpoints
│   └── middleware/     # Server middleware
├── utils/              # Auto-imported utilities
├── app.vue             # Root component
├── nuxt.config.ts      # Nuxt configuration
└── package.json
```

## App Entry Point

```vue
<!-- app.vue -->
<template>
  <NuxtLayout>
    <NuxtPage />
  </NuxtLayout>
</template>
```

- `<NuxtLayout>` - Renders the current layout
- `<NuxtPage>` - Renders the current page

## Configuration

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // Enable devtools
  devtools: { enabled: true },
  
  // SSR mode (default: true)
  ssr: true,
  
  // Modules
  modules: [
    '@nuxt/ui',
    '@pinia/nuxt',
    '@nuxtjs/tailwindcss'
  ],
  
  // Runtime config
  runtimeConfig: {
    // Private (server only)
    apiSecret: '',
    // Public (client & server)
    public: {
      apiBase: '/api'
    }
  },
  
  // App config
  app: {
    head: {
      title: 'My Nuxt App',
      meta: [
        { name: 'description', content: 'My amazing app' }
      ]
    }
  },
  
  // Route rules
  routeRules: {
    '/': { prerender: true },
    '/admin/**': { ssr: false }
  }
})
```

## Auto-Imports

Nuxt auto-imports:

```vue
<script setup>
// Vue APIs - auto-imported
const count = ref(0)
const double = computed(() => count.value * 2)

// Nuxt composables - auto-imported
const route = useRoute()
const { data } = await useFetch('/api/users')

// Your composables from /composables - auto-imported
const { user } = useAuth()
</script>
```

No imports needed for:
- Vue reactivity (`ref`, `computed`, `watch`)
- Vue lifecycle (`onMounted`, `onUnmounted`)
- Nuxt composables (`useFetch`, `useRoute`, `useState`)
- Your composables in `/composables`
- Your utilities in `/utils`
- Components in `/components`

## Running Nuxt

```bash
# Development
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Generate static site
npm run generate
```

## Resources

- [Nuxt Documentation](https://nuxt.com/docs) — Official Nuxt 3 documentation

---

> 📘 *This lesson is part of the [Vue Server-Side Rendering & Nuxt](https://stanza.dev/courses/vue-ssr-nuxt) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*