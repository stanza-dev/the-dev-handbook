---
source_course: "vue-component-architecture"
source_lesson: "vue-component-architecture-component-v-model"
---

# Component v-model for Two-Way Binding

`v-model` on components creates two-way binding between parent and child. It's syntactic sugar for a prop + event combination.

## How v-model Works on Components

When you write:

```vue
<CustomInput v-model="searchText" />
```

It expands to:

```vue
<CustomInput
  :model-value="searchText"
  @update:model-value="searchText = $event"
/>
```

## Implementing v-model

The child component must:
1. Accept a `modelValue` prop
2. Emit `update:modelValue` events

```vue
<!-- CustomInput.vue -->
<script setup lang="ts">
const props = defineProps<{
  modelValue: string
}>()

const emit = defineEmits<{
  'update:modelValue': [value: string]
}>()

function handleInput(event: Event) {
  const target = event.target as HTMLInputElement
  emit('update:modelValue', target.value)
}
</script>

<template>
  <input
    :value="modelValue"
    @input="handleInput"
    class="custom-input"
  />
</template>
```

## Using defineModel() (Vue 3.4+)

Simplified syntax with the `defineModel` macro:

```vue
<!-- CustomInput.vue -->
<script setup lang="ts">
const model = defineModel<string>()
</script>

<template>
  <input v-model="model" class="custom-input" />
</template>
```

`defineModel()` returns a ref that:
- Syncs with the parent's value
- Auto-emits on changes
- Works with `v-model` in templates

## Named v-model

Use a custom name instead of `modelValue`:

```vue
<!-- Parent -->
<UserName
  v-model:first-name="firstName"
  v-model:last-name="lastName"
/>
```

```vue
<!-- UserName.vue -->
<script setup lang="ts">
const firstName = defineModel<string>('firstName')
const lastName = defineModel<string>('lastName')
</script>

<template>
  <input v-model="firstName" placeholder="First name" />
  <input v-model="lastName" placeholder="Last name" />
</template>
```

## Multiple v-model Bindings

Components can have multiple v-model bindings:

```vue
<!-- Parent -->
<UserForm
  v-model:email="email"
  v-model:password="password"
  v-model:remember="rememberMe"
/>
```

```vue
<!-- UserForm.vue -->
<script setup lang="ts">
const email = defineModel<string>('email')
const password = defineModel<string>('password')
const remember = defineModel<boolean>('remember')
</script>

<template>
  <form>
    <input v-model="email" type="email" placeholder="Email" />
    <input v-model="password" type="password" placeholder="Password" />
    <label>
      <input v-model="remember" type="checkbox" />
      Remember me
    </label>
  </form>
</template>
```

## v-model Modifiers

### Built-in Modifiers

Parents can use modifiers:

```vue
<CustomInput v-model.trim="text" />
<CustomInput v-model.number="quantity" />
<CustomInput v-model.lazy="message" />
```

### Custom Modifiers

Access modifiers in the child:

```vue
<script setup lang="ts">
const [model, modifiers] = defineModel<string>({
  set(value) {
    // Apply capitalize modifier
    if (modifiers.capitalize && value) {
      return value.charAt(0).toUpperCase() + value.slice(1)
    }
    return value
  }
})
</script>

<template>
  <input v-model="model" />
</template>
```

```vue
<!-- Parent -->
<CustomInput v-model.capitalize="name" />
```

## Practical Example: Rating Component

```vue
<!-- StarRating.vue -->
<script setup lang="ts">
const rating = defineModel<number>({ default: 0 })

const props = withDefaults(defineProps<{
  maxStars?: number
  readonly?: boolean
}>(), {
  maxStars: 5,
  readonly: false
})

function setRating(value: number) {
  if (!props.readonly) {
    rating.value = value
  }
}
</script>

<template>
  <div class="star-rating" :class="{ readonly }">
    <button
      v-for="star in maxStars"
      :key="star"
      type="button"
      :class="{ filled: star <= rating }"
      @click="setRating(star)"
      :disabled="readonly"
    >
      ★
    </button>
  </div>
</template>

<style scoped>
.star-rating button {
  background: none;
  border: none;
  font-size: 1.5rem;
  color: #ddd;
  cursor: pointer;
}

.star-rating button.filled {
  color: #f5c518;
}

.star-rating.readonly button {
  cursor: default;
}
</style>
```

```vue
<!-- Usage -->
<template>
  <StarRating v-model="productRating" />
  <StarRating v-model="reviewRating" :max-stars="10" />
  <StarRating :model-value="averageRating" readonly />
</template>
```

## Resources

- [Component v-model](https://vuejs.org/guide/components/v-model.html) — Official documentation on component v-model

---

> 📘 *This lesson is part of the [Vue Component Architecture](https://stanza.dev/courses/vue-component-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*