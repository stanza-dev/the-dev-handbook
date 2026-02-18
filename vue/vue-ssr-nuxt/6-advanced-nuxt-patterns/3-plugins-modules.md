---
source_course: "vue-ssr-nuxt"
source_lesson: "vue-ssr-nuxt-plugins-modules"
---

# Plugins and Modules

Extend Nuxt with plugins for Vue functionality and modules for Nuxt features.

## Plugins

Plugins run when the Vue app is created.

### Basic Plugin

```typescript
// plugins/my-plugin.ts
export default defineNuxtPlugin(() => {
  console.log('Plugin loaded!')
})
```

### Providing Helpers

```typescript
// plugins/format.ts
export default defineNuxtPlugin(() => {
  return {
    provide: {
      formatDate: (date: Date) => {
        return new Intl.DateTimeFormat('en-US').format(date)
      },
      formatCurrency: (amount: number) => {
        return new Intl.NumberFormat('en-US', {
          style: 'currency',
          currency: 'USD'
        }).format(amount)
      }
    }
  }
})
```

Usage:

```vue
<script setup>
const { $formatDate, $formatCurrency } = useNuxtApp()

const formattedDate = $formatDate(new Date())
const price = $formatCurrency(29.99)
</script>
```

### Vue Directive Plugin

```typescript
// plugins/directives.ts
export default defineNuxtPlugin((nuxtApp) => {
  nuxtApp.vueApp.directive('focus', {
    mounted(el) {
      el.focus()
    }
  })
  
  nuxtApp.vueApp.directive('click-outside', {
    mounted(el, binding) {
      el._clickOutside = (event: Event) => {
        if (!el.contains(event.target)) {
          binding.value()
        }
      }
      document.addEventListener('click', el._clickOutside)
    },
    unmounted(el) {
      document.removeEventListener('click', el._clickOutside)
    }
  })
})
```

### Third-Party Library Plugin

```typescript
// plugins/dayjs.ts
import dayjs from 'dayjs'
import relativeTime from 'dayjs/plugin/relativeTime'

export default defineNuxtPlugin(() => {
  dayjs.extend(relativeTime)
  
  return {
    provide: {
      dayjs
    }
  }
})
```

### Client/Server Only Plugins

```typescript
// plugins/client-analytics.client.ts
export default defineNuxtPlugin(() => {
  // Only runs in browser
  window.analytics.init('key')
})

// plugins/server-logger.server.ts
export default defineNuxtPlugin(() => {
  // Only runs on server
  console.log('Server started')
})
```

## Modules

Modules extend Nuxt's core functionality.

### Installing Modules

```bash
npm install @nuxtjs/tailwindcss @pinia/nuxt
```

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: [
    '@nuxtjs/tailwindcss',
    '@pinia/nuxt',
    '@nuxt/image',
    '@vueuse/nuxt'
  ]
})
```

### Popular Modules

```typescript
export default defineNuxtConfig({
  modules: [
    // Styling
    '@nuxtjs/tailwindcss',
    '@nuxt/ui',
    
    // State
    '@pinia/nuxt',
    
    // Images
    '@nuxt/image',
    
    // Content
    '@nuxt/content',
    
    // SEO
    '@nuxtjs/seo',
    
    // Auth
    '@sidebase/nuxt-auth',
    
    // Utils
    '@vueuse/nuxt'
  ]
})
```

### Creating a Local Module

```typescript
// modules/my-module.ts
import { defineNuxtModule, addPlugin, createResolver } from '@nuxt/kit'

export default defineNuxtModule({
  meta: {
    name: 'my-module',
    configKey: 'myModule'
  },
  defaults: {
    enabled: true
  },
  setup(options, nuxt) {
    if (!options.enabled) return
    
    const { resolve } = createResolver(import.meta.url)
    
    // Add a plugin
    addPlugin(resolve('./runtime/plugin'))
    
    // Add composables
    addImportsDir(resolve('./runtime/composables'))
    
    // Add components
    addComponentsDir({ path: resolve('./runtime/components') })
  }
})
```

## Resources

- [Nuxt Plugins](https://nuxt.com/docs/guide/directory-structure/plugins) — Nuxt plugins documentation

---

> 📘 *This lesson is part of the [Vue Server-Side Rendering & Nuxt](https://stanza.dev/courses/vue-ssr-nuxt) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*