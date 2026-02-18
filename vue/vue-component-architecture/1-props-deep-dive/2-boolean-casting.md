---
source_course: "vue-component-architecture"
source_lesson: "vue-component-architecture-boolean-casting"
---

# Boolean Casting and Props Best Practices

Boolean props have special behavior in Vue that mirrors HTML boolean attributes. Understanding this is crucial for building intuitive component APIs.

## Boolean Casting Rules

When a prop is declared with `Boolean` type:

```vue
<script setup>
const props = defineProps({
  disabled: Boolean,
  visible: Boolean,
  loading: Boolean
})
</script>
```

These usages are equivalent:

```vue
<!-- Presence = true -->
<MyButton disabled />
<MyButton :disabled="true" />

<!-- Absence = false -->
<MyButton />
<MyButton :disabled="false" />
```

This mimics native HTML behavior like `<button disabled>`.

## Mixed Types with Boolean

When combining Boolean with other types, order matters:

```vue
<script setup>
const props = defineProps({
  // Boolean before String - boolean casting applies
  disabled: [Boolean, String],
  
  // String before Boolean - no boolean casting
  label: [String, Boolean]
})
</script>
```

```vue
<!-- With [Boolean, String] -->
<MyComponent disabled />       <!-- disabled = true -->
<MyComponent disabled="" />    <!-- disabled = true -->

<!-- With [String, Boolean] -->
<MyComponent label />          <!-- label = '' (empty string) -->
<MyComponent label="" />       <!-- label = '' (empty string) -->
```

## One-Way Data Flow

Props form a **one-way-down binding**:

- Parent prop changes flow down to child
- Child should never mutate props

```vue
<script setup>
const props = defineProps(['initialValue'])

// ❌ NEVER do this
function handleClick() {
  props.initialValue = 'new value'  // Warning!
}
</script>
```

### Pattern 1: Local Data from Prop

Use the prop as an initial value:

```vue
<script setup>
import { ref } from 'vue'

const props = defineProps(['initialCounter'])

// Create local state initialized from prop
const counter = ref(props.initialCounter)

// Now counter is independent and can be modified
function increment() {
  counter.value++
}
</script>
```

### Pattern 2: Computed from Prop

Derive values without mutation:

```vue
<script setup>
import { computed } from 'vue'

const props = defineProps(['size'])

// Transform the prop, don't mutate it
const normalizedSize = computed(() => {
  return props.size.trim().toLowerCase()
})
</script>
```

### Pattern 3: Emit Changes to Parent

Ask the parent to update:

```vue
<script setup>
const props = defineProps(['modelValue'])
const emit = defineEmits(['update:modelValue'])

function handleInput(event) {
  // Don't mutate - emit!
  emit('update:modelValue', event.target.value)
}
</script>

<template>
  <input :value="modelValue" @input="handleInput" />
</template>
```

## Props Naming Conventions

### camelCase in JavaScript

```vue
<script setup>
const props = defineProps({
  greetingMessage: String,
  itemCount: Number,
  isVisible: Boolean
})
</script>
```

### kebab-case in Templates

```vue
<template>
  <MyComponent
    greeting-message="Hello"
    :item-count="5"
    is-visible
  />
</template>
```

Vue automatically converts between them.

## Object Props: Passing Multiple Values

Bind an object to pass all its properties as props:

```vue
<script setup>
import { reactive } from 'vue'

const post = reactive({
  id: 1,
  title: 'My Post',
  author: 'John',
  publishedAt: '2024-01-15'
})
</script>

<template>
  <!-- Pass individual props -->
  <BlogPost
    :id="post.id"
    :title="post.title"
    :author="post.author"
    :published-at="post.publishedAt"
  />
  
  <!-- Or spread all props at once -->
  <BlogPost v-bind="post" />
</template>
```

## Best Practices Summary

1. **Use TypeScript** for compile-time type checking
2. **Always validate** props in reusable components
3. **Use factory functions** for object/array defaults
4. **Never mutate props** - use local state or emit events
5. **Name booleans positively** (`isVisible` not `isHidden`)
6. **Document required props** clearly
7. **Provide sensible defaults** when possible

## Resources

- [One-Way Data Flow](https://vuejs.org/guide/components/props.html#one-way-data-flow) — Understanding Vue's one-way data flow

---

> 📘 *This lesson is part of the [Vue Component Architecture](https://stanza.dev/courses/vue-component-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*