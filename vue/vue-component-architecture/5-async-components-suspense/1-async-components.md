---
source_course: "vue-component-architecture"
source_lesson: "vue-component-architecture-async-components"
---

# Lazy Loading with Async Components

Async components load only when needed, reducing initial bundle size and improving application performance.

## Basic Async Component

Use `defineAsyncComponent` to create lazy-loaded components:

```typescript
import { defineAsyncComponent } from 'vue'

// Basic usage
const AsyncModal = defineAsyncComponent(() => 
  import('./components/Modal.vue')
)

// With loading and error states
const AsyncUserProfile = defineAsyncComponent({
  loader: () => import('./components/UserProfile.vue'),
  loadingComponent: LoadingSpinner,
  errorComponent: ErrorDisplay,
  delay: 200,         // Show loading after 200ms
  timeout: 10000      // Timeout after 10 seconds
})
```

## Using Async Components

```vue
<script setup lang="ts">
import { defineAsyncComponent, ref } from 'vue'

// Component only loads when modal opens
const AsyncModal = defineAsyncComponent(() => 
  import('./components/HeavyModal.vue')
)

const showModal = ref(false)
</script>

<template>
  <button @click="showModal = true">Open Modal</button>
  
  <!-- Component loads when v-if becomes true -->
  <AsyncModal v-if="showModal" @close="showModal = false" />
</template>
```

## Loading and Error States

Handle loading and error scenarios:

```vue
<script setup lang="ts">
import { defineAsyncComponent, h } from 'vue'

// Loading component
const LoadingSpinner = {
  render() {
    return h('div', { class: 'loading' }, 'Loading...')
  }
}

// Error component
const ErrorComponent = {
  props: ['error'],
  render() {
    return h('div', { class: 'error' }, `Error: ${this.error.message}`)
  }
}

const AsyncDashboard = defineAsyncComponent({
  loader: () => import('./components/Dashboard.vue'),
  loadingComponent: LoadingSpinner,
  errorComponent: ErrorComponent,
  delay: 200,       // Don't show loading for quick loads
  timeout: 30000,   // Fail after 30 seconds
  onError(error, retry, fail, attempts) {
    if (error.message.includes('network') && attempts < 3) {
      retry()  // Retry on network errors
    } else {
      fail()   // Give up
    }
  }
})
</script>
```

## Route-Based Code Splitting

Combine with Vue Router for route-level splitting:

```typescript
// router/index.ts
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/',
      component: () => import('@/views/HomeView.vue')
    },
    {
      path: '/dashboard',
      component: () => import('@/views/DashboardView.vue')
    },
    {
      path: '/admin',
      component: () => import('@/views/AdminView.vue')
    }
  ]
})
```

## Conditional Loading Pattern

```vue
<script setup lang="ts">
import { defineAsyncComponent, shallowRef, markRaw } from 'vue'

type ChartType = 'line' | 'bar' | 'pie'

const chartComponents = {
  line: defineAsyncComponent(() => import('./charts/LineChart.vue')),
  bar: defineAsyncComponent(() => import('./charts/BarChart.vue')),
  pie: defineAsyncComponent(() => import('./charts/PieChart.vue'))
}

const currentChart = shallowRef(markRaw(chartComponents.line))

function setChartType(type: ChartType) {
  currentChart.value = markRaw(chartComponents[type])
}
</script>

<template>
  <div class="chart-selector">
    <button @click="setChartType('line')">Line</button>
    <button @click="setChartType('bar')">Bar</button>
    <button @click="setChartType('pie')">Pie</button>
  </div>
  
  <component :is="currentChart" :data="chartData" />
</template>
```

## Bundle Analysis

Async components create separate chunks. Use Vite's build analysis:

```bash
npx vite build --mode development
npx vite-bundle-visualizer
```

You'll see chunks like:
- `Modal.vue` → `Modal-abc123.js`
- `Dashboard.vue` → `Dashboard-def456.js`

These load on demand rather than in the initial bundle.

## Resources

- [Async Components](https://vuejs.org/guide/components/async.html) — Official Vue documentation on async components

---

> 📘 *This lesson is part of the [Vue Component Architecture](https://stanza.dev/courses/vue-component-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*