---
source_course: "vue-component-architecture"
source_lesson: "vue-component-architecture-scoped-slots"
---

# Scoped Slots for Data Passing

Scoped slots let child components pass data back to the parent's slot content. This enables powerful patterns where the child handles logic while the parent controls rendering.

## Basic Scoped Slot

Child passes data via slot props:

```vue
<!-- UserList.vue -->
<script setup lang="ts">
import { ref, onMounted } from 'vue'

type User = { id: number; name: string; email: string }

const users = ref<User[]>([])
const loading = ref(true)

onMounted(async () => {
  const response = await fetch('/api/users')
  users.value = await response.json()
  loading.value = false
})
</script>

<template>
  <div class="user-list">
    <div v-if="loading">Loading...</div>
    <template v-else>
      <!-- Pass data to parent via slot props -->
      <slot 
        v-for="user in users" 
        :key="user.id"
        :user="user"
        :index="users.indexOf(user)"
      ></slot>
    </template>
  </div>
</template>
```

Parent receives data in slot:

```vue
<!-- Parent -->
<UserList v-slot="{ user, index }">
  <div class="user-card">
    <span>{{ index + 1 }}.</span>
    <strong>{{ user.name }}</strong>
    <span>{{ user.email }}</span>
  </div>
</UserList>
```

## Named Scoped Slots

Combine named slots with slot props:

```vue
<!-- DataTable.vue -->
<script setup lang="ts">
type Column = { key: string; label: string }

const props = defineProps<{
  columns: Column[]
  data: Record<string, any>[]
}>()
</script>

<template>
  <table>
    <thead>
      <tr>
        <th v-for="col in columns" :key="col.key">
          <slot :name="`header-${col.key}`" :column="col">
            {{ col.label }}
          </slot>
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(row, rowIndex) in data" :key="rowIndex">
        <td v-for="col in columns" :key="col.key">
          <slot 
            :name="`cell-${col.key}`" 
            :value="row[col.key]"
            :row="row"
            :rowIndex="rowIndex"
          >
            {{ row[col.key] }}
          </slot>
        </td>
      </tr>
    </tbody>
  </table>
</template>
```

```vue
<!-- Parent -->
<DataTable :columns="columns" :data="users">
  <!-- Custom header for 'name' column -->
  <template #header-name="{ column }">
    <strong>👤 {{ column.label }}</strong>
  </template>
  
  <!-- Custom cell for 'status' column -->
  <template #cell-status="{ value }">
    <span :class="['badge', value]">{{ value }}</span>
  </template>
  
  <!-- Custom cell for 'actions' column -->
  <template #cell-actions="{ row }">
    <button @click="edit(row)">Edit</button>
    <button @click="remove(row)">Delete</button>
  </template>
</DataTable>
```

## Renderless Components

Components that provide only logic, no rendering:

```vue
<!-- MouseTracker.vue -->
<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const x = ref(0)
const y = ref(0)

function update(event: MouseEvent) {
  x.value = event.clientX
  y.value = event.clientY
}

onMounted(() => window.addEventListener('mousemove', update))
onUnmounted(() => window.removeEventListener('mousemove', update))
</script>

<template>
  <!-- No visual output, just passes data -->
  <slot :x="x" :y="y"></slot>
</template>
```

```vue
<!-- Parent controls rendering -->
<MouseTracker v-slot="{ x, y }">
  <div class="cursor-display">
    Mouse: {{ x }}, {{ y }}
  </div>
</MouseTracker>

<MouseTracker v-slot="{ x, y }">
  <div 
    class="follower"
    :style="{ left: x + 'px', top: y + 'px' }"
  ></div>
</MouseTracker>
```

## Fetch Component Pattern

```vue
<!-- FetchData.vue -->
<script setup lang="ts">
import { ref, watchEffect } from 'vue'

const props = defineProps<{
  url: string
}>()

const data = ref(null)
const error = ref<Error | null>(null)
const loading = ref(true)

watchEffect(async () => {
  loading.value = true
  error.value = null
  
  try {
    const response = await fetch(props.url)
    if (!response.ok) throw new Error('Fetch failed')
    data.value = await response.json()
  } catch (e) {
    error.value = e as Error
  } finally {
    loading.value = false
  }
})
</script>

<template>
  <slot 
    :data="data" 
    :error="error" 
    :loading="loading"
    :refetch="() => { /* trigger refetch */ }"
  ></slot>
</template>
```

```vue
<!-- Parent -->
<FetchData url="/api/users" v-slot="{ data, error, loading }">
  <div v-if="loading">Loading...</div>
  <div v-else-if="error">Error: {{ error.message }}</div>
  <ul v-else>
    <li v-for="user in data" :key="user.id">{{ user.name }}</li>
  </ul>
</FetchData>
```

## Slot Props TypeScript

Type your slot props:

```vue
<script setup lang="ts">
import { defineSlots } from 'vue'

type User = { id: number; name: string }

defineSlots<{
  default(props: { user: User; index: number }): any
  header(props: { title: string }): any
  empty(props: {}): any
}>()
</script>
```

## Resources

- [Scoped Slots](https://vuejs.org/guide/components/slots.html#scoped-slots) — Official documentation on scoped slots

---

> 📘 *This lesson is part of the [Vue Component Architecture](https://stanza.dev/courses/vue-component-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*