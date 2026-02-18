---
source_course: "vue-animations"
source_lesson: "vue-animations-transition-modes-dynamic"
---

# Transition Modes and Dynamic Transitions

Control how entering and leaving elements coordinate, and dynamically change transitions.

## Transition Modes

By default, enter and leave happen simultaneously. Use `mode` to change this:

```vue
<script setup>
import { ref } from 'vue'

const currentView = ref('A')
</script>

<template>
  <button @click="currentView = currentView === 'A' ? 'B' : 'A'">
    Toggle View
  </button>
  
  <!-- Default: both animate simultaneously -->
  <Transition name="fade">
    <component :is="currentView === 'A' ? ViewA : ViewB" :key="currentView" />
  </Transition>
  
  <!-- out-in: leave first, then enter -->
  <Transition name="fade" mode="out-in">
    <component :is="currentView === 'A' ? ViewA : ViewB" :key="currentView" />
  </Transition>
  
  <!-- in-out: enter first, then leave -->
  <Transition name="fade" mode="in-out">
    <component :is="currentView === 'A' ? ViewA : ViewB" :key="currentView" />
  </Transition>
</template>
```

### Mode Comparison

| Mode | Behavior | Best For |
|------|----------|----------|
| (default) | Simultaneous | Cross-fades |
| `out-in` | Leave → Enter | Tab switching |
| `in-out` | Enter → Leave | Overlapping reveals |

## Dynamic Transitions

Change the transition based on state:

```vue
<script setup>
import { ref, computed } from 'vue'

const direction = ref<'left' | 'right'>('right')
const currentStep = ref(1)

const transitionName = computed(() => 
  direction.value === 'right' ? 'slide-right' : 'slide-left'
)

function next() {
  direction.value = 'right'
  currentStep.value++
}

function prev() {
  direction.value = 'left'
  currentStep.value--
}
</script>

<template>
  <div class="wizard">
    <button @click="prev" :disabled="currentStep <= 1">Previous</button>
    <button @click="next" :disabled="currentStep >= 3">Next</button>
    
    <Transition :name="transitionName" mode="out-in">
      <Step1 v-if="currentStep === 1" key="1" />
      <Step2 v-else-if="currentStep === 2" key="2" />
      <Step3 v-else key="3" />
    </Transition>
  </div>
</template>

<style>
.slide-right-enter-active,
.slide-right-leave-active,
.slide-left-enter-active,
.slide-left-leave-active {
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.slide-right-enter-from {
  transform: translateX(100%);
  opacity: 0;
}
.slide-right-leave-to {
  transform: translateX(-100%);
  opacity: 0;
}

.slide-left-enter-from {
  transform: translateX(-100%);
  opacity: 0;
}
.slide-left-leave-to {
  transform: translateX(100%);
  opacity: 0;
}
</style>
```

## Reusable Transition Components

```vue
<!-- FadeTransition.vue -->
<template>
  <Transition
    name="fade"
    :mode="mode"
    :appear="appear"
    @after-enter="$emit('after-enter')"
    @after-leave="$emit('after-leave')"
  >
    <slot />
  </Transition>
</template>

<script setup lang="ts">
defineProps<{
  mode?: 'in-out' | 'out-in'
  appear?: boolean
}>()

defineEmits<{
  'after-enter': []
  'after-leave': []
}>()
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
```

Usage:

```vue
<FadeTransition mode="out-in" appear>
  <div v-if="show">Content</div>
</FadeTransition>
```

## Resources

- [Transition Modes](https://vuejs.org/guide/built-ins/transition.html#transition-modes) — Vue transition modes documentation

---

> 📘 *This lesson is part of the [Vue Animations & Transitions](https://stanza.dev/courses/vue-animations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*