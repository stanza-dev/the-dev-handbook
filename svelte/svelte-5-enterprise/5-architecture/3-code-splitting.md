---
source_course: "svelte-5-enterprise"
source_lesson: "svelte-5-enterprise-code-splitting"
---

# Optimizing Large Applications

## SvelteKit's Automatic Code Splitting

SvelteKit splits code by route automatically:

```
routes/
├── +page.svelte         → bundle for /
├── about/
│   └── +page.svelte     → bundle for /about
└── dashboard/
    └── +page.svelte     → bundle for /dashboard
```

Users only download code for pages they visit!

## Dynamic Imports

```svelte
<script>
  let HeavyComponent;
  let showChart = $state(false);
  
  async function loadChart() {
    showChart = true;
    const module = await import('./HeavyChart.svelte');
    HeavyComponent = module.default;
  }
</script>

<button onclick={loadChart}>Show Chart</button>

{#if showChart && HeavyComponent}
  <svelte:component this={HeavyComponent} />
{/if}
```

## Preloading

```svelte
<!-- Preload on hover -->
<a href="/dashboard" data-sveltekit-preload-data="hover">
  Dashboard
</a>

<!-- Preload on viewport entry -->
<a href="/about" data-sveltekit-preload-data="viewport">
  About
</a>
```

## Bundle Analysis

```bash
# Install analyzer
npm install -D rollup-plugin-visualizer

# Add to vite.config.js
import { visualizer } from 'rollup-plugin-visualizer';

plugins: [
  visualizer({
    filename: 'stats.html',
    open: true
  })
]

# Build and analyze
npm run build
```

## Common Optimizations

**1. Lazy load heavy dependencies**
```javascript
// Don't import at top level
// import { heavyLibrary } from 'heavy-lib';

// Import when needed
const { heavyLibrary } = await import('heavy-lib');
```

**2. Tree-shake imports**
```javascript
// ❌ Imports everything
import * as lodash from 'lodash';

// ✅ Only imports what you use
import { debounce } from 'lodash-es';
```

**3. Image optimization**
```svelte
<!-- Use SvelteKit's image handling -->
import { enhance } from '$app/forms';

<!-- Or vite-imagetools -->
<img src="./photo.jpg?w=400" alt="..." />
```

📖 [SvelteKit performance](https://svelte.dev/docs/kit/performance)

---

> 📘 *This lesson is part of the [Enterprise Svelte: Testing & Architecture](https://stanza.dev/courses/svelte-5-enterprise) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*