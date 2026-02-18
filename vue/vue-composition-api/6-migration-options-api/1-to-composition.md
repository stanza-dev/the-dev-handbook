---
source_course: "vue-composition-api"
source_lesson: "vue-composition-api-options-to-composition"
---

# Options API to Composition API Migration

Learn to convert existing Options API components to the Composition API.

## Side-by-Side Comparison

### Options API

```vue
<script>
export default {
  props: {
    userId: {
      type: Number,
      required: true
    }
  },
  
  emits: ['update', 'delete'],
  
  data() {
    return {
      user: null,
      loading: false,
      error: null
    }
  },
  
  computed: {
    fullName() {
      return this.user ? `${this.user.firstName} ${this.user.lastName}` : ''
    },
    isAdmin() {
      return this.user?.role === 'admin'
    }
  },
  
  watch: {
    userId: {
      immediate: true,
      handler(newId) {
        this.fetchUser(newId)
      }
    }
  },
  
  methods: {
    async fetchUser(id) {
      this.loading = true
      this.error = null
      try {
        const response = await fetch(`/api/users/${id}`)
        this.user = await response.json()
      } catch (e) {
        this.error = e.message
      } finally {
        this.loading = false
      }
    },
    handleUpdate() {
      this.$emit('update', this.user)
    },
    handleDelete() {
      this.$emit('delete', this.userId)
    }
  },
  
  mounted() {
    console.log('Component mounted')
  }
}
</script>
```

### Composition API Equivalent

```vue
<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'

// Props
const props = defineProps<{
  userId: number
}>()

// Emits
const emit = defineEmits<{
  update: [user: User]
  delete: [userId: number]
}>()

// Data (reactive state)
const user = ref<User | null>(null)
const loading = ref(false)
const error = ref<string | null>(null)

// Computed
const fullName = computed(() =>
  user.value ? `${user.value.firstName} ${user.value.lastName}` : ''
)
const isAdmin = computed(() => user.value?.role === 'admin')

// Methods
async function fetchUser(id: number) {
  loading.value = true
  error.value = null
  try {
    const response = await fetch(`/api/users/${id}`)
    user.value = await response.json()
  } catch (e) {
    error.value = (e as Error).message
  } finally {
    loading.value = false
  }
}

function handleUpdate() {
  if (user.value) {
    emit('update', user.value)
  }
}

function handleDelete() {
  emit('delete', props.userId)
}

// Watch
watch(
  () => props.userId,
  (newId) => fetchUser(newId),
  { immediate: true }
)

// Lifecycle
onMounted(() => {
  console.log('Component mounted')
})
</script>
```

## Migration Mapping

| Options API | Composition API |
|-------------|----------------|
| `props: {}` | `defineProps<{}>()` |
| `emits: []` | `defineEmits<{}>()` |
| `data()` | `ref()` / `reactive()` |
| `computed: {}` | `computed()` |
| `watch: {}` | `watch()` |
| `methods: {}` | Regular functions |
| `mounted()` | `onMounted()` |
| `this.xxx` | Direct access / `.value` |
| `this.$emit()` | `emit()` |

## Extracting to Composables

During migration, extract reusable logic:

```typescript
// composables/useUser.ts
export function useUser(userId: Ref<number>) {
  const user = ref<User | null>(null)
  const loading = ref(false)
  const error = ref<string | null>(null)
  
  async function fetchUser() {
    loading.value = true
    error.value = null
    try {
      const response = await fetch(`/api/users/${userId.value}`)
      user.value = await response.json()
    } catch (e) {
      error.value = (e as Error).message
    } finally {
      loading.value = false
    }
  }
  
  watch(userId, fetchUser, { immediate: true })
  
  const fullName = computed(() =>
    user.value ? `${user.value.firstName} ${user.value.lastName}` : ''
  )
  
  return { user, loading, error, fullName, refetch: fetchUser }
}
```

Component becomes much simpler:

```vue
<script setup lang="ts">
import { useUser } from '@/composables/useUser'

const props = defineProps<{ userId: number }>()
const emit = defineEmits<{
  update: [user: User]
  delete: [userId: number]
}>()

const { user, loading, error, fullName } = useUser(toRef(props, 'userId'))

function handleUpdate() {
  if (user.value) emit('update', user.value)
}
</script>
```

## Resources

- [Composition API FAQ](https://vuejs.org/guide/extras/composition-api-faq.html) — Vue Composition API comparison and FAQ

---

> 📘 *This lesson is part of the [Vue Composition API & Composables](https://stanza.dev/courses/vue-composition-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*