---
source_course: "vue-typescript"
source_lesson: "vue-typescript-generic-components"
---

# Generic Components

Vue 3.3+ supports generic components, enabling powerful patterns for reusable, type-safe components.

## Basic Generic Component

```vue
<script setup lang="ts" generic="T">
defineProps<{
  items: T[]
  selected?: T
}>()

const emit = defineEmits<{
  select: [item: T]
}>()
</script>

<template>
  <ul>
    <li 
      v-for="item in items" 
      :key="String(item)"
      @click="emit('select', item)"
    >
      <slot :item="item" />
    </li>
  </ul>
</template>
```

```vue
<!-- Usage -->
<script setup lang="ts">
interface User {
  id: number
  name: string
}

const users: User[] = [...]
const selectedUser = ref<User | undefined>()

function handleSelect(user: User) {
  // user is typed as User!
  selectedUser.value = user
}
</script>

<template>
  <GenericList 
    :items="users" 
    :selected="selectedUser"
    @select="handleSelect"
  >
    <template #default="{ item }">
      <!-- item is typed as User -->
      {{ item.name }}
    </template>
  </GenericList>
</template>
```

## Multiple Type Parameters

```vue
<script setup lang="ts" generic="T, K extends keyof T">
defineProps<{
  items: T[]
  labelKey: K
  valueKey: K
}>()
</script>
```

## Generic with Constraints

```vue
<script setup lang="ts" generic="T extends { id: number }">
defineProps<{
  items: T[]
  selectedId?: number
}>()

// T must have an id property
const getById = (id: number) => 
  props.items.find(item => item.id === id)
</script>
```

## Select Component Example

```vue
<!-- Select.vue -->
<script setup lang="ts" generic="T">
import { computed } from 'vue'

interface Props {
  options: T[]
  modelValue?: T
  labelKey?: keyof T
  placeholder?: string
}

const props = withDefaults(defineProps<Props>(), {
  placeholder: 'Select an option'
})

const emit = defineEmits<{
  'update:modelValue': [value: T | undefined]
}>()

const getLabel = (option: T): string => {
  if (props.labelKey) {
    return String(option[props.labelKey])
  }
  return String(option)
}
</script>

<template>
  <select 
    :value="modelValue" 
    @change="emit('update:modelValue', options[($event.target as HTMLSelectElement).selectedIndex])"
  >
    <option disabled value="">{{ placeholder }}</option>
    <option v-for="(option, index) in options" :key="index" :value="option">
      {{ getLabel(option) }}
    </option>
  </select>
</template>
```

## DataTable Generic Component

```vue
<script setup lang="ts" generic="T extends Record<string, unknown>">
interface Column<T> {
  key: keyof T
  label: string
  sortable?: boolean
  render?: (value: T[keyof T], row: T) => string
}

defineProps<{
  data: T[]
  columns: Column<T>[]
  loading?: boolean
}>()

const emit = defineEmits<{
  'row-click': [row: T]
  sort: [key: keyof T, direction: 'asc' | 'desc']
}>()

defineSlots<{
  cell(props: { column: Column<T>; row: T; value: T[keyof T] }): any
  empty(): any
  loading(): any
}>()
</script>

<template>
  <table>
    <thead>
      <tr>
        <th v-for="column in columns" :key="String(column.key)">
          {{ column.label }}
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-if="loading">
        <td :colspan="columns.length">
          <slot name="loading">Loading...</slot>
        </td>
      </tr>
      <tr v-else-if="data.length === 0">
        <td :colspan="columns.length">
          <slot name="empty">No data</slot>
        </td>
      </tr>
      <tr 
        v-else
        v-for="(row, index) in data" 
        :key="index"
        @click="emit('row-click', row)"
      >
        <td v-for="column in columns" :key="String(column.key)">
          <slot 
            name="cell" 
            :column="column" 
            :row="row" 
            :value="row[column.key]"
          >
            {{ column.render ? column.render(row[column.key], row) : row[column.key] }}
          </slot>
        </td>
      </tr>
    </tbody>
  </table>
</template>
```

```vue
<!-- Usage -->
<script setup lang="ts">
interface Product {
  id: number
  name: string
  price: number
  category: string
}

const products: Product[] = [...]

const columns = [
  { key: 'name' as const, label: 'Name', sortable: true },
  { key: 'price' as const, label: 'Price', render: (v) => `$${v}` },
  { key: 'category' as const, label: 'Category' }
]
</script>

<template>
  <DataTable 
    :data="products" 
    :columns="columns"
    @row-click="(product) => console.log(product.name)"
  />
</template>
```

## Resources

- [Generic Components](https://vuejs.org/api/sfc-script-setup.html#generics) — Official documentation on generic components

---

> 📘 *This lesson is part of the [TypeScript with Vue](https://stanza.dev/courses/vue-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*