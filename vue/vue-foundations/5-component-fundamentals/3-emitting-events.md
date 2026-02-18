---
source_course: "vue-foundations"
source_lesson: "vue-foundations-emitting-events"
---

# Emitting Events to Parents

While props flow data **down** from parent to child, events flow **up** from child to parent. This is how children communicate changes back to their parents.

## Defining Events

Use `defineEmits()` to declare what events a component can emit:

```vue
<!-- src/components/CounterButton.vue -->
<script setup lang="ts">
const emit = defineEmits<{
  increment: []  // No payload
  decrement: []  // No payload
  change: [newValue: number]  // With payload
}>()

function handleIncrement() {
  emit('increment')
}

function handleDecrement() {
  emit('decrement')
}
</script>

<template>
  <div class="counter-buttons">
    <button @click="handleDecrement">-</button>
    <button @click="handleIncrement">+</button>
  </div>
</template>
```

## Listening to Events

Parents listen with `@event-name`:

```vue
<!-- Parent component -->
<script setup lang="ts">
import CounterButton from './components/CounterButton.vue'
import { ref } from 'vue'

const count = ref(0)

function handleIncrement() {
  count.value++
}

function handleDecrement() {
  count.value--
}
</script>

<template>
  <div>
    <p>Count: {{ count }}</p>
    <CounterButton 
      @increment="handleIncrement" 
      @decrement="handleDecrement" 
    />
  </div>
</template>
```

## Events with Payloads

Pass data with emitted events:

```vue
<!-- src/components/SearchBox.vue -->
<script setup lang="ts">
import { ref } from 'vue'

const emit = defineEmits<{
  search: [query: string]
  clear: []
}>()

const query = ref('')

function handleSubmit() {
  if (query.value.trim()) {
    emit('search', query.value.trim())
  }
}

function handleClear() {
  query.value = ''
  emit('clear')
}
</script>

<template>
  <form @submit.prevent="handleSubmit" class="search-box">
    <input 
      v-model="query" 
      placeholder="Search..." 
    />
    <button type="submit">Search</button>
    <button type="button" @click="handleClear">Clear</button>
  </form>
</template>
```

```vue
<!-- Parent component -->
<script setup lang="ts">
import SearchBox from './components/SearchBox.vue'
import { ref } from 'vue'

const searchResults = ref<string[]>([])

function handleSearch(query: string) {
  console.log('Searching for:', query)
  // Perform search...
}

function handleClear() {
  searchResults.value = []
}
</script>

<template>
  <SearchBox 
    @search="handleSearch" 
    @clear="handleClear" 
  />
</template>
```

## Event Naming Convention

Use **kebab-case** for event names in templates:

```vue
<!-- Child component -->
<script setup lang="ts">
const emit = defineEmits<{
  itemSelected: [id: number]  // camelCase in TypeScript
  updateValue: [value: string]
}>()

emit('itemSelected', 42)
emit('updateValue', 'hello')
</script>

<!-- Parent component template -->
<template>
  <!-- Use kebab-case when listening -->
  <ChildComponent 
    @item-selected="handleSelect"
    @update-value="handleUpdate"
  />
</template>
```

## Inline Event Handlers

For simple cases, use inline handlers:

```vue
<template>
  <!-- Access payload with $event -->
  <SearchBox @search="searchQuery = $event" />
  
  <!-- Or inline function -->
  <SearchBox @search="(q) => performSearch(q)" />
  
  <!-- Multiple arguments -->
  <DataTable @sort="(column, direction) => sortBy(column, direction)" />
</template>
```

## Complete Example: Todo Item

```vue
<!-- src/components/TodoItem.vue -->
<script setup lang="ts">
type Todo = {
  id: number
  text: string
  done: boolean
}

const props = defineProps<{
  todo: Todo
}>()

const emit = defineEmits<{
  toggle: [id: number]
  delete: [id: number]
  edit: [id: number, newText: string]
}>()

import { ref } from 'vue'

const isEditing = ref(false)
const editText = ref('')

function startEdit() {
  editText.value = props.todo.text
  isEditing.value = true
}

function saveEdit() {
  if (editText.value.trim()) {
    emit('edit', props.todo.id, editText.value.trim())
  }
  isEditing.value = false
}

function cancelEdit() {
  isEditing.value = false
}
</script>

<template>
  <li class="todo-item" :class="{ done: todo.done }">
    <input 
      type="checkbox" 
      :checked="todo.done"
      @change="emit('toggle', todo.id)"
    />
    
    <template v-if="isEditing">
      <input 
        v-model="editText" 
        @keyup.enter="saveEdit"
        @keyup.esc="cancelEdit"
      />
      <button @click="saveEdit">Save</button>
      <button @click="cancelEdit">Cancel</button>
    </template>
    
    <template v-else>
      <span @dblclick="startEdit">{{ todo.text }}</span>
      <button @click="emit('delete', todo.id)">Delete</button>
    </template>
  </li>
</template>

<style scoped>
.todo-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem;
}

.done span {
  text-decoration: line-through;
  color: #888;
}
</style>
```

```vue
<!-- Parent: TodoList.vue -->
<script setup lang="ts">
import TodoItem from './TodoItem.vue'
import { ref } from 'vue'

const todos = ref([
  { id: 1, text: 'Learn Vue', done: true },
  { id: 2, text: 'Build an app', done: false },
])

function toggleTodo(id: number) {
  const todo = todos.value.find(t => t.id === id)
  if (todo) todo.done = !todo.done
}

function deleteTodo(id: number) {
  todos.value = todos.value.filter(t => t.id !== id)
}

function editTodo(id: number, newText: string) {
  const todo = todos.value.find(t => t.id === id)
  if (todo) todo.text = newText
}
</script>

<template>
  <ul>
    <TodoItem
      v-for="todo in todos"
      :key="todo.id"
      :todo="todo"
      @toggle="toggleTodo"
      @delete="deleteTodo"
      @edit="editTodo"
    />
  </ul>
</template>
```

## Resources

- [Component Events](https://vuejs.org/guide/components/events.html) — Official Vue documentation on emitting events

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*