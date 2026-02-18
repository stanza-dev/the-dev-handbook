---
source_course: "vue-animations"
source_lesson: "vue-animations-transition-group-basics"
---

# TransitionGroup Basics

`<TransitionGroup>` animates multiple elements in a list, including additions, removals, and reordering.

## Key Differences from Transition

| Transition | TransitionGroup |
|------------|----------------|
| Single child | Multiple children |
| No wrapper element | Renders wrapper (default: `<span>`) |
| v-if/v-show | v-for with :key |

## Basic Usage

```vue
<script setup>
import { ref } from 'vue'

const items = ref([1, 2, 3])
let id = 4

function add() {
  items.value.push(id++)
}

function remove(item: number) {
  items.value = items.value.filter(i => i !== item)
}
</script>

<template>
  <button @click="add">Add</button>
  
  <TransitionGroup name="list" tag="ul">
    <li v-for="item in items" :key="item" @click="remove(item)">
      {{ item }}
    </li>
  </TransitionGroup>
</template>

<style>
.list-enter-active,
.list-leave-active {
  transition: all 0.3s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(-30px);
}
</style>
```

## The tag Prop

```vue
<!-- Renders as <ul> -->
<TransitionGroup tag="ul">
  <li v-for="item in items" :key="item">{{ item }}</li>
</TransitionGroup>

<!-- Renders as <div> -->
<TransitionGroup tag="div" class="grid">
  <div v-for="item in items" :key="item">{{ item }}</div>
</TransitionGroup>

<!-- No wrapper (Vue 3.5+) -->
<TransitionGroup :tag="null">
  <div v-for="item in items" :key="item">{{ item }}</div>
</TransitionGroup>
```

## Move Transitions

Animate items moving to new positions:

```vue
<TransitionGroup name="list" tag="ul">
  <li v-for="item in items" :key="item">{{ item }}</li>
</TransitionGroup>

<style>
/* Enter and leave animations */
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

/* IMPORTANT: Move animation */
.list-move {
  transition: transform 0.5s ease;
}

/* Fix leaving items taking space */
.list-leave-active {
  position: absolute;
}
</style>
```

## Staggered Animations

Delay animations based on index:

```vue
<script setup>
function onBeforeEnter(el: HTMLElement) {
  el.style.opacity = '0'
  el.style.transform = 'translateY(20px)'
}

function onEnter(el: HTMLElement, done: () => void) {
  const delay = parseInt(el.dataset.index || '0') * 100
  
  setTimeout(() => {
    el.style.transition = 'all 0.3s ease'
    el.style.opacity = '1'
    el.style.transform = 'translateY(0)'
    
    setTimeout(done, 300)
  }, delay)
}
</script>

<template>
  <TransitionGroup 
    tag="ul"
    @before-enter="onBeforeEnter"
    @enter="onEnter"
  >
    <li 
      v-for="(item, index) in items" 
      :key="item.id"
      :data-index="index"
    >
      {{ item.name }}
    </li>
  </TransitionGroup>
</template>
```

## Shuffle Animation

```vue
<script setup>
import { ref } from 'vue'

const items = ref([1, 2, 3, 4, 5, 6, 7, 8, 9])

function shuffle() {
  items.value = [...items.value].sort(() => Math.random() - 0.5)
}
</script>

<template>
  <button @click="shuffle">Shuffle</button>
  
  <TransitionGroup name="shuffle" tag="div" class="grid">
    <div v-for="item in items" :key="item" class="item">
      {{ item }}
    </div>
  </TransitionGroup>
</template>

<style>
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

.item {
  padding: 20px;
  background: #42b883;
  color: white;
  text-align: center;
}

.shuffle-move {
  transition: transform 0.5s ease;
}
</style>
```

## Filter/Sort Animation

```vue
<script setup>
import { ref, computed } from 'vue'

const items = ref([
  { id: 1, name: 'Apple', category: 'fruit' },
  { id: 2, name: 'Carrot', category: 'vegetable' },
  { id: 3, name: 'Banana', category: 'fruit' },
  { id: 4, name: 'Broccoli', category: 'vegetable' },
])

const filter = ref('all')

const filteredItems = computed(() => {
  if (filter.value === 'all') return items.value
  return items.value.filter(i => i.category === filter.value)
})
</script>

<template>
  <select v-model="filter">
    <option value="all">All</option>
    <option value="fruit">Fruits</option>
    <option value="vegetable">Vegetables</option>
  </select>
  
  <TransitionGroup name="filter" tag="ul">
    <li v-for="item in filteredItems" :key="item.id">
      {{ item.name }}
    </li>
  </TransitionGroup>
</template>

<style>
.filter-enter-active,
.filter-leave-active {
  transition: all 0.3s ease;
}

.filter-enter-from,
.filter-leave-to {
  opacity: 0;
  transform: scale(0.9);
}

.filter-move {
  transition: transform 0.3s ease;
}

.filter-leave-active {
  position: absolute;
}
</style>
```

## Resources

- [TransitionGroup](https://vuejs.org/guide/built-ins/transition-group.html) — Official TransitionGroup documentation

---

> 📘 *This lesson is part of the [Vue Animations & Transitions](https://stanza.dev/courses/vue-animations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*