---
source_course: "vue-composition-api"
source_lesson: "vue-composition-api-compiler-macros"
---

# Compiler Macros: defineProps, defineEmits, and More

`<script setup>` provides special compiler macros that don't need to be imported. They're transformed at compile time.

## defineProps

Declare the props your component accepts:

```vue
<script setup lang="ts">
// Runtime declaration
const props = defineProps({
  title: String,
  count: {
    type: Number,
    required: true
  },
  disabled: {
    type: Boolean,
    default: false
  }
})

// Type-based declaration (recommended with TypeScript)
const props = defineProps<{
  title?: string
  count: number
  disabled?: boolean
}>()

// Access props
console.log(props.count)
</script>
```

### withDefaults for Type-Based Props

```vue
<script setup lang="ts">
type Props = {
  msg?: string
  labels?: string[]
  count?: number
}

const props = withDefaults(defineProps<Props>(), {
  msg: 'Hello',
  labels: () => ['one', 'two'],  // Factory for non-primitives
  count: 0
})
</script>
```

## defineEmits

Declare events your component can emit:

```vue
<script setup lang="ts">
// Runtime declaration
const emit = defineEmits(['update', 'delete'])

// Type-based declaration
const emit = defineEmits<{
  (e: 'update', id: number, value: string): void
  (e: 'delete', id: number): void
}>()

// Vue 3.3+ shorthand
const emit = defineEmits<{
  update: [id: number, value: string]
  delete: [id: number]
}>()

// Emit events
emit('update', 1, 'new value')
emit('delete', 1)
</script>
```

## defineModel (Vue 3.4+)

Simplify two-way binding for `v-model`:

```vue
<!-- Child component -->
<script setup>
// Replaces props + emit pattern for v-model
const modelValue = defineModel()

// With options
const count = defineModel('count', { 
  type: Number, 
  default: 0 
})

// TypeScript
const modelValue = defineModel<string>()
const count = defineModel<number>('count', { required: true })
</script>

<template>
  <input :value="modelValue" @input="modelValue = $event.target.value" />
</template>

<!-- Parent -->
<template>
  <MyInput v-model="text" v-model:count="counter" />
</template>
```

## defineExpose

Expose properties to parent via template refs:

```vue
<!-- Child -->
<script setup>
import { ref } from 'vue'

const count = ref(0)
const privateData = ref('secret')

function reset() {
  count.value = 0
}

// Only these are accessible via template ref
defineExpose({
  count,
  reset
})
</script>

<!-- Parent -->
<script setup>
import { ref, onMounted } from 'vue'
import Child from './Child.vue'

const childRef = ref<InstanceType<typeof Child> | null>(null)

onMounted(() => {
  console.log(childRef.value?.count)  // Works
  childRef.value?.reset()             // Works
  // childRef.value?.privateData      // undefined
})
</script>

<template>
  <Child ref="childRef" />
</template>
```

## defineOptions (Vue 3.3+)

Set component options without a separate `<script>` block:

```vue
<script setup>
defineOptions({
  name: 'CustomName',
  inheritAttrs: false,
  customOptions: {
    /* ... */
  }
})
</script>
```

## defineSlots (Vue 3.3+)

Provide type hints for slots:

```vue
<script setup lang="ts">
const slots = defineSlots<{
  default(props: { message: string }): any
  header(props: { title: string }): any
  footer(): any
}>()
</script>
```

## useSlots and useAttrs

Access slots and fallthrough attributes:

```vue
<script setup>
import { useSlots, useAttrs } from 'vue'

const slots = useSlots()
const attrs = useAttrs()

// Check if slot exists
if (slots.header) {
  console.log('Header slot provided')
}

// Access fallthrough attrs
console.log(attrs.class, attrs.style)
</script>
```

## Resources

- [defineProps and defineEmits](https://vuejs.org/api/sfc-script-setup.html#defineprops-defineemits) — Official documentation for compiler macros

---

> 📘 *This lesson is part of the [Vue Composition API & Composables](https://stanza.dev/courses/vue-composition-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*