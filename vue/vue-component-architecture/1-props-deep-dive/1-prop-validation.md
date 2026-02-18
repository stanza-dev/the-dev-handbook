---
source_course: "vue-component-architecture"
source_lesson: "vue-component-architecture-prop-validation"
---

# Advanced Prop Validation

Props are the primary mechanism for passing data from parent to child components. In this lesson, we'll explore advanced validation techniques to create robust, self-documenting components.

## Runtime vs Type-Based Declaration

Vue offers two ways to declare props:

### Type-Based (TypeScript)

```vue
<script setup lang="ts">
type Status = 'pending' | 'active' | 'completed'

const props = defineProps<{
  title: string
  count: number
  status: Status
  items?: string[]
}>()
</script>
```

### Runtime Declaration

```vue
<script setup>
const props = defineProps({
  title: {
    type: String,
    required: true
  },
  count: {
    type: Number,
    default: 0
  },
  status: {
    type: String,
    validator: (value) => ['pending', 'active', 'completed'].includes(value)
  },
  items: {
    type: Array,
    default: () => []
  }
})
</script>
```

## Prop Type Options

Vue supports these native constructors:

- `String`
- `Number`
- `Boolean`
- `Array`
- `Object`
- `Date`
- `Function`
- `Symbol`

### Multiple Types

```vue
<script setup>
const props = defineProps({
  // Can be String OR Number
  id: [String, Number],
  
  // With full options
  value: {
    type: [String, Number],
    required: true
  }
})
</script>
```

## Custom Validators

Validators return `true` for valid values, `false` for invalid:

```vue
<script setup>
const props = defineProps({
  // Enum-like validation
  size: {
    type: String,
    default: 'medium',
    validator: (value) => {
      return ['small', 'medium', 'large', 'xl'].includes(value)
    }
  },
  
  // Range validation
  percentage: {
    type: Number,
    default: 0,
    validator: (value) => value >= 0 && value <= 100
  },
  
  // Email format validation
  email: {
    type: String,
    validator: (value) => {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
      return emailRegex.test(value)
    }
  },
  
  // Object shape validation
  user: {
    type: Object,
    validator: (value) => {
      return value.id && value.name && typeof value.id === 'number'
    }
  }
})
</script>
```

## Default Values

### Primitive Defaults

```vue
<script setup>
const props = defineProps({
  message: {
    type: String,
    default: 'Hello World'
  },
  count: {
    type: Number,
    default: 0
  },
  isActive: {
    type: Boolean,
    default: false
  }
})
</script>
```

### Object/Array Defaults (Factory Functions)

Always use factory functions for objects and arrays to avoid shared references:

```vue
<script setup>
const props = defineProps({
  // ❌ Wrong - shared reference between instances
  // items: { type: Array, default: [] },
  
  // ✅ Correct - new array for each instance
  items: {
    type: Array,
    default: () => []
  },
  
  config: {
    type: Object,
    default: () => ({
      theme: 'light',
      showHeader: true,
      maxItems: 10
    })
  },
  
  user: {
    type: Object,
    default: () => ({
      name: 'Guest',
      role: 'viewer'
    })
  }
})
</script>
```

## TypeScript with withDefaults

Combine TypeScript types with default values:

```vue
<script setup lang="ts">
type Size = 'sm' | 'md' | 'lg'
type Theme = 'light' | 'dark'

type Props = {
  title: string
  size?: Size
  theme?: Theme
  items?: string[]
  config?: {
    showHeader: boolean
    maxItems: number
  }
}

const props = withDefaults(defineProps<Props>(), {
  size: 'md',
  theme: 'light',
  items: () => [],
  config: () => ({
    showHeader: true,
    maxItems: 10
  })
})
</script>
```

## Required Props

Mark props that must be provided:

```vue
<script setup lang="ts">
// TypeScript - no ? means required
const props = defineProps<{
  id: number          // Required
  name: string        // Required  
  description?: string // Optional
}>()
</script>
```

```vue
<script setup>
// Runtime declaration
const props = defineProps({
  id: {
    type: Number,
    required: true
  },
  name: {
    type: String,
    required: true
  },
  description: String  // Optional by default
})
</script>
```

## Validation Warnings

In development, Vue warns about validation failures:

```
[Vue warn]: Invalid prop: custom validator check failed for prop "size".
[Vue warn]: Missing required prop: "id"
[Vue warn]: Invalid prop: type check failed for prop "count". Expected Number, got String.
```

These warnings only appear in development mode, not in production builds.

## Resources

- [Props Documentation](https://vuejs.org/guide/components/props.html) — Official Vue documentation on component props

---

> 📘 *This lesson is part of the [Vue Component Architecture](https://stanza.dev/courses/vue-component-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*