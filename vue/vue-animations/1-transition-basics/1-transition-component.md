---
source_course: "vue-animations"
source_lesson: "vue-animations-transition-component"
---

# The Transition Component

Vue's `<Transition>` component lets you animate elements entering and leaving the DOM.

## Basic Usage

```vue
<script setup>
import { ref } from 'vue'

const show = ref(true)
</script>

<template>
  <button @click="show = !show">Toggle</button>
  
  <Transition name="fade">
    <div v-if="show">Hello!</div>
  </Transition>
</template>

<style>
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

## Transition Classes

Vue applies classes at different stages:

```
Enter transition:
v-enter-from → v-enter-active → v-enter-to

Leave transition:
v-leave-from → v-leave-active → v-leave-to
```

| Class | When Applied |
|-------|-------------|
| `v-enter-from` | Before element is inserted |
| `v-enter-active` | During entire enter phase |
| `v-enter-to` | After element is inserted |
| `v-leave-from` | When leave starts |
| `v-leave-active` | During entire leave phase |
| `v-leave-to` | After leave ends |

## Named Transitions

The `name` prop prefixes the classes:

```vue
<Transition name="slide">
  <div v-if="show">Content</div>
</Transition>

<style>
/* slide-enter-from, slide-enter-active, etc. */
.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s ease;
}

.slide-enter-from,
.slide-leave-to {
  transform: translateX(-100%);
}
</style>
```

## CSS Transitions vs Animations

### CSS Transitions

```css
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
```

### CSS Animations

```css
.bounce-enter-active {
  animation: bounce-in 0.5s;
}

.bounce-leave-active {
  animation: bounce-in 0.5s reverse;
}

@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.2);
  }
  100% {
    transform: scale(1);
  }
}
```

## Custom Class Names

Use with animation libraries like Animate.css:

```vue
<Transition
  enter-active-class="animate__animated animate__fadeIn"
  leave-active-class="animate__animated animate__fadeOut"
>
  <div v-if="show">Animated!</div>
</Transition>
```

## Transition Modes

Control enter/leave timing:

```vue
<!-- Default: both animations run simultaneously -->
<Transition>
  <component :is="currentComponent" />
</Transition>

<!-- out-in: wait for leave to finish before enter -->
<Transition mode="out-in">
  <component :is="currentComponent" :key="currentComponent" />
</Transition>

<!-- in-out: wait for enter to finish before leave -->
<Transition mode="in-out">
  <component :is="currentComponent" :key="currentComponent" />
</Transition>
```

## Transition with v-show

```vue
<Transition name="fade">
  <!-- Works with v-show too -->
  <div v-show="show">Always in DOM, just hidden</div>
</Transition>
```

## Important Rules

1. Only ONE direct child element
2. Child must have `v-if`, `v-show`, or be a dynamic component
3. Child must not be always-rendered

```vue
<!-- ❌ Wrong: multiple children -->
<Transition>
  <div v-if="a">A</div>
  <div v-if="b">B</div>
</Transition>

<!-- ✅ Correct: single child -->
<Transition>
  <div v-if="show">Content</div>
</Transition>

<!-- ✅ Correct: mutually exclusive -->
<Transition>
  <div v-if="type === 'a'" key="a">A</div>
  <div v-else-if="type === 'b'" key="b">B</div>
</Transition>
```

## Resources

- [Transition](https://vuejs.org/guide/built-ins/transition.html) — Official Vue Transition documentation

---

> 📘 *This lesson is part of the [Vue Animations & Transitions](https://stanza.dev/courses/vue-animations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*