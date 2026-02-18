---
source_course: "vue-animations"
source_lesson: "vue-animations-common-animations"
---

# Common Animation Patterns

Here are ready-to-use animation patterns for common UI needs.

## Fade

```vue
<Transition name="fade">
  <div v-if="show">Content</div>
</Transition>

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

## Slide

```vue
<!-- Slide from left -->
<Transition name="slide-left">
  <div v-if="show">Content</div>
</Transition>

<style>
.slide-left-enter-active,
.slide-left-leave-active {
  transition: transform 0.3s ease;
}

.slide-left-enter-from {
  transform: translateX(-100%);
}

.slide-left-leave-to {
  transform: translateX(100%);
}
</style>
```

## Scale

```vue
<Transition name="scale">
  <div v-if="show">Content</div>
</Transition>

<style>
.scale-enter-active,
.scale-leave-active {
  transition: transform 0.2s ease, opacity 0.2s ease;
}

.scale-enter-from,
.scale-leave-to {
  transform: scale(0.9);
  opacity: 0;
}
</style>
```

## Slide + Fade (Dropdown)

```vue
<Transition name="dropdown">
  <ul v-if="isOpen" class="menu">
    <li>Item 1</li>
    <li>Item 2</li>
  </ul>
</Transition>

<style>
.dropdown-enter-active,
.dropdown-leave-active {
  transition: opacity 0.2s ease, transform 0.2s ease;
  transform-origin: top;
}

.dropdown-enter-from,
.dropdown-leave-to {
  opacity: 0;
  transform: scaleY(0.9) translateY(-10px);
}
</style>
```

## Modal/Dialog

```vue
<Transition name="modal">
  <div v-if="showModal" class="modal-overlay" @click.self="close">
    <div class="modal-content">
      <slot />
    </div>
  </div>
</Transition>

<style>
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.3s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-active .modal-content,
.modal-leave-active .modal-content {
  transition: transform 0.3s ease;
}

.modal-enter-from .modal-content,
.modal-leave-to .modal-content {
  transform: scale(0.9) translateY(20px);
}
</style>
```

## Toast Notification

```vue
<Transition name="toast">
  <div v-if="showToast" class="toast">
    {{ message }}
  </div>
</Transition>

<style>
.toast {
  position: fixed;
  bottom: 20px;
  right: 20px;
  padding: 12px 24px;
  background: #333;
  color: white;
  border-radius: 8px;
}

.toast-enter-active,
.toast-leave-active {
  transition: all 0.3s ease;
}

.toast-enter-from {
  opacity: 0;
  transform: translateX(100%);
}

.toast-leave-to {
  opacity: 0;
  transform: translateY(20px);
}
</style>
```

## Page Transitions

```vue
<router-view v-slot="{ Component, route }">
  <Transition name="page" mode="out-in">
    <component :is="Component" :key="route.path" />
  </Transition>
</router-view>

<style>
.page-enter-active,
.page-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.page-enter-from {
  opacity: 0;
  transform: translateY(20px);
}

.page-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}
</style>
```

## Reusable Transition Component

```vue
<!-- FadeTransition.vue -->
<script setup>
defineProps<{
  duration?: number
}>()
</script>

<template>
  <Transition name="fade" :style="{ '--duration': duration + 'ms' }">
    <slot />
  </Transition>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity var(--duration, 300ms) ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>

<!-- Usage -->
<FadeTransition :duration="500">
  <div v-if="show">Content</div>
</FadeTransition>
```

## Resources

- [CSS Transitions](https://vuejs.org/guide/built-ins/transition.html#css-based-transitions) — CSS-based transitions in Vue

---

> 📘 *This lesson is part of the [Vue Animations & Transitions](https://stanza.dev/courses/vue-animations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*