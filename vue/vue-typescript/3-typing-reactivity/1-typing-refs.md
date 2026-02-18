---
source_course: "vue-typescript"
source_lesson: "vue-typescript-typing-refs"
---

# Typing Refs and Reactive

Vue's reactivity system works great with TypeScript, with type inference handling most cases automatically.

## ref() Typing

### Automatic Inference

```typescript
import { ref } from 'vue'

// Type inferred from initial value
const count = ref(0)           // Ref<number>
const name = ref('Alice')      // Ref<string>
const isActive = ref(true)     // Ref<boolean>
const items = ref(['a', 'b'])  // Ref<string[]>
```

### Explicit Typing

```typescript
import { ref, type Ref } from 'vue'

// Generic type parameter
const count = ref<number>(0)
const name = ref<string>('')

// Union types
const value = ref<string | number>('')

// Nullable types
const user = ref<User | null>(null)
const error = ref<Error | undefined>(undefined)

// Using Ref type
const items: Ref<string[]> = ref([])
```

### Complex Types

```typescript
import { ref } from 'vue'

interface User {
  id: number
  name: string
  email: string
}

interface ApiResponse<T> {
  data: T
  status: number
  message: string
}

// Object type
const user = ref<User | null>(null)

// Array of objects
const users = ref<User[]>([])

// Generic response
const response = ref<ApiResponse<User[]> | null>(null)

// Map
const userMap = ref<Map<number, User>>(new Map())

// Set
const selectedIds = ref<Set<number>>(new Set())
```

## reactive() Typing

### Automatic Inference

```typescript
import { reactive } from 'vue'

// Type inferred from object structure
const state = reactive({
  count: 0,
  name: 'Vue',
  items: [] as string[]  // Type assertion for empty arrays
})

// state.count is number
// state.name is string
// state.items is string[]
```

### Interface Typing

```typescript
import { reactive } from 'vue'

interface State {
  user: User | null
  loading: boolean
  error: string | null
  items: Item[]
}

const state = reactive<State>({
  user: null,
  loading: false,
  error: null,
  items: []
})

// Or with initial values matching interface
const state: State = reactive({
  user: null,
  loading: false,
  error: null,
  items: []
})
```

## computed() Typing

### Automatic Inference

```typescript
import { ref, computed } from 'vue'

const count = ref(0)

// Return type inferred
const double = computed(() => count.value * 2)  // ComputedRef<number>
const message = computed(() => `Count: ${count.value}`)  // ComputedRef<string>
```

### Explicit Return Type

```typescript
import { ref, computed } from 'vue'

const items = ref<Item[]>([])

// Explicit when inference is complex
const stats = computed<{ total: number; average: number }>(() => {
  const total = items.value.reduce((sum, item) => sum + item.value, 0)
  return {
    total,
    average: items.value.length ? total / items.value.length : 0
  }
})
```

### Writable Computed

```typescript
import { ref, computed, type WritableComputedRef } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

const fullName: WritableComputedRef<string> = computed({
  get: () => `${firstName.value} ${lastName.value}`,
  set: (value: string) => {
    const parts = value.split(' ')
    firstName.value = parts[0]
    lastName.value = parts.slice(1).join(' ')
  }
})
```

## Template Refs Typing

```typescript
import { ref, onMounted } from 'vue'

// DOM element ref
const inputRef = ref<HTMLInputElement | null>(null)
const divRef = ref<HTMLDivElement | null>(null)
const canvasRef = ref<HTMLCanvasElement | null>(null)

onMounted(() => {
  // TypeScript knows the type
  inputRef.value?.focus()
  const ctx = canvasRef.value?.getContext('2d')
})
```

### Component Refs

```typescript
import { ref } from 'vue'
import ChildComponent from './ChildComponent.vue'

// Get component instance type
const childRef = ref<InstanceType<typeof ChildComponent> | null>(null)

// Access exposed methods/properties
childRef.value?.someExposedMethod()
```

## shallowRef Typing

```typescript
import { shallowRef, triggerRef } from 'vue'

interface HeavyObject {
  data: number[]
  metadata: Record<string, string>
}

const heavy = shallowRef<HeavyObject>({
  data: [],
  metadata: {}
})

// Replace entire value
heavy.value = { data: [1, 2, 3], metadata: {} }

// Or mutate and trigger
heavy.value.data.push(4)
triggerRef(heavy)
```

## Resources

- [Typing Reactive](https://vuejs.org/guide/typescript/composition-api.html#typing-reactive) — Official guide on typing reactivity

---

> 📘 *This lesson is part of the [TypeScript with Vue](https://stanza.dev/courses/vue-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*