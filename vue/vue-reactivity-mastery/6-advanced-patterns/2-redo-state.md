---
source_course: "vue-reactivity-mastery"
source_lesson: "vue-reactivity-mastery-undo-redo-state"
---

# Undo/Redo State Management

Implement undo/redo functionality using Vue's reactivity system.

## Basic History Stack

```typescript
import { ref, computed, watch, type Ref } from 'vue'

export function useHistory<T>(source: Ref<T>, maxHistory = 50) {
  const history = ref<T[]>([]) as Ref<T[]>
  const future = ref<T[]>([]) as Ref<T[]>
  let ignoreUpdate = false
  
  // Initialize with current value
  history.value.push(structuredClone(source.value))
  
  // Track changes
  watch(
    source,
    (newValue) => {
      if (ignoreUpdate) {
        ignoreUpdate = false
        return
      }
      
      // Clear redo stack on new change
      future.value = []
      
      // Add to history
      history.value.push(structuredClone(newValue))
      
      // Limit history size
      if (history.value.length > maxHistory) {
        history.value.shift()
      }
    },
    { deep: true }
  )
  
  const canUndo = computed(() => history.value.length > 1)
  const canRedo = computed(() => future.value.length > 0)
  
  function undo() {
    if (!canUndo.value) return
    
    const current = history.value.pop()!
    future.value.push(current)
    
    ignoreUpdate = true
    source.value = structuredClone(history.value[history.value.length - 1])
  }
  
  function redo() {
    if (!canRedo.value) return
    
    const next = future.value.pop()!
    history.value.push(next)
    
    ignoreUpdate = true
    source.value = structuredClone(next)
  }
  
  function clear() {
    history.value = [structuredClone(source.value)]
    future.value = []
  }
  
  return {
    canUndo,
    canRedo,
    undo,
    redo,
    clear,
    historyLength: computed(() => history.value.length),
    futureLength: computed(() => future.value.length)
  }
}
```

## Usage Example: Drawing App

```vue
<script setup>
import { reactive } from 'vue'
import { useHistory } from '@/composables/useHistory'

const canvas = reactive({
  shapes: [],
  selectedId: null
})

const { canUndo, canRedo, undo, redo } = useHistory(
  toRef(() => canvas.shapes)
)

function addShape(shape) {
  canvas.shapes.push({ id: Date.now(), ...shape })
}

function deleteSelected() {
  if (canvas.selectedId) {
    canvas.shapes = canvas.shapes.filter(s => s.id !== canvas.selectedId)
    canvas.selectedId = null
  }
}
</script>

<template>
  <div class="toolbar">
    <button @click="undo" :disabled="!canUndo">
      ↶ Undo
    </button>
    <button @click="redo" :disabled="!canRedo">
      ↷ Redo
    </button>
  </div>
  
  <!-- Canvas with shapes -->
</template>
```

## Keyboard Shortcuts

```typescript
import { onMounted, onUnmounted } from 'vue'

export function useHistoryKeyboard(history: ReturnType<typeof useHistory>) {
  function handleKeydown(e: KeyboardEvent) {
    if ((e.ctrlKey || e.metaKey) && e.key === 'z') {
      e.preventDefault()
      if (e.shiftKey) {
        history.redo()
      } else {
        history.undo()
      }
    }
    if ((e.ctrlKey || e.metaKey) && e.key === 'y') {
      e.preventDefault()
      history.redo()
    }
  }
  
  onMounted(() => {
    document.addEventListener('keydown', handleKeydown)
  })
  
  onUnmounted(() => {
    document.removeEventListener('keydown', handleKeydown)
  })
}
```

## Resources

- [Reactivity API](https://vuejs.org/api/reactivity-core.html) — Vue reactivity core API

---

> 📘 *This lesson is part of the [Vue Reactivity Mastery](https://stanza.dev/courses/vue-reactivity-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*