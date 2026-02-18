---
source_course: "vue-component-architecture"
source_lesson: "vue-component-architecture-attribute-inheritance"
---

# Understanding Attribute Inheritance

Fallthrough attributes are attributes or event listeners passed to a component that aren't declared as props or emits. They automatically "fall through" to the component's root element.

## Basic Fallthrough

```vue
<!-- MyButton.vue -->
<template>
  <button class="btn">
    <slot></slot>
  </button>
</template>
```

```vue
<!-- Parent -->
<MyButton class="primary" id="submit-btn" @click="handleClick">
  Submit
</MyButton>
```

Rendered HTML:

```html
<button class="btn primary" id="submit-btn">
  Submit
</button>
```

The `class`, `id`, and `@click` automatically apply to the root `<button>`.

## Class and Style Merging

Classes and styles merge intelligently:

```vue
<!-- MyInput.vue -->
<template>
  <input class="base-input" style="border: 1px solid gray" />
</template>
```

```vue
<!-- Parent -->
<MyInput class="large" style="padding: 1rem" />
```

Result:

```html
<input 
  class="base-input large" 
  style="border: 1px solid gray; padding: 1rem" 
/>
```

## v-on Listener Inheritance

Event listeners also fall through:

```vue
<!-- MyButton.vue -->
<template>
  <button><slot></slot></button>
</template>
```

```vue
<!-- Parent -->
<MyButton @click="onClick" @focus="onFocus">
  Click me
</MyButton>
```

Both `@click` and `@focus` will fire when the button is interacted with.

## Disabling Attribute Inheritance

Sometimes you don't want automatic inheritance:

```vue
<script setup lang="ts">
defineOptions({
  inheritAttrs: false
})
</script>

<template>
  <div class="wrapper">
    <!-- Attributes won't auto-apply to wrapper -->
    <input class="input" />
  </div>
</template>
```

## Accessing Fallthrough Attributes

Use `useAttrs()` to access and manually apply attributes:

```vue
<script setup lang="ts">
import { useAttrs } from 'vue'

defineOptions({
  inheritAttrs: false
})

const attrs = useAttrs()
</script>

<template>
  <div class="input-wrapper">
    <label>{{ label }}</label>
    <!-- Apply attrs to the input, not the wrapper -->
    <input v-bind="attrs" class="input" />
  </div>
</template>
```

```vue
<!-- Parent -->
<MyInput 
  label="Email" 
  type="email" 
  placeholder="Enter email" 
  @focus="onFocus"
/>
```

Now `type`, `placeholder`, and `@focus` apply to the `<input>`, not the wrapper div.

## Multi-Root Components

Components with multiple root elements don't auto-inherit:

```vue
<!-- MultiRoot.vue -->
<template>
  <header>Header</header>
  <main>Main</main>
  <footer>Footer</footer>
</template>
```

```vue
<!-- Parent -->
<MultiRoot class="styled" />  <!-- Warning! No auto-inheritance -->
```

You must explicitly bind attrs:

```vue
<script setup>
import { useAttrs } from 'vue'

const attrs = useAttrs()
</script>

<template>
  <header>Header</header>
  <main v-bind="attrs">Main</main>
  <footer>Footer</footer>
</template>
```

## Practical Example: Form Input Component

```vue
<!-- FormInput.vue -->
<script setup lang="ts">
import { useAttrs, computed } from 'vue'

defineOptions({
  inheritAttrs: false
})

const props = defineProps<{
  label: string
  modelValue: string
  error?: string
}>()

const emit = defineEmits<{
  'update:modelValue': [value: string]
}>()

const attrs = useAttrs()

// Separate class from other attrs
const inputClass = computed(() => {
  const classes = ['input']
  if (props.error) classes.push('input-error')
  if (attrs.class) classes.push(attrs.class as string)
  return classes.join(' ')
})

const inputAttrs = computed(() => {
  const { class: _, ...rest } = attrs
  return rest
})
</script>

<template>
  <div class="form-group">
    <label :for="attrs.id || undefined">{{ label }}</label>
    <input
      :value="modelValue"
      @input="emit('update:modelValue', ($event.target as HTMLInputElement).value)"
      :class="inputClass"
      v-bind="inputAttrs"
    />
    <span v-if="error" class="error">{{ error }}</span>
  </div>
</template>
```

```vue
<!-- Usage -->
<FormInput
  v-model="email"
  label="Email Address"
  type="email"
  id="email-input"
  placeholder="you@example.com"
  class="large"
  required
  @focus="handleFocus"
  :error="emailError"
/>
```

## Best Practices

1. **Keep inheritAttrs: true** for simple wrapper components
2. **Disable and manually apply** when you need control over where attrs go
3. **Document** which attributes your component forwards
4. **Be mindful of events** - they also fall through
5. **Use TypeScript** to type your expected attrs when possible

## Resources

- [Fallthrough Attributes](https://vuejs.org/guide/components/attrs.html) — Official Vue documentation on attribute inheritance

---

> 📘 *This lesson is part of the [Vue Component Architecture](https://stanza.dev/courses/vue-component-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*