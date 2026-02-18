---
source_course: "vue-animations"
source_lesson: "vue-animations-js-hooks"
---

# JavaScript Transition Hooks

JavaScript hooks give you full control over animations, enabling integration with libraries like GSAP, anime.js, or custom animations.

## Available Hooks

```vue
<Transition
  @before-enter="onBeforeEnter"
  @enter="onEnter"
  @after-enter="onAfterEnter"
  @enter-cancelled="onEnterCancelled"
  @before-leave="onBeforeLeave"
  @leave="onLeave"
  @after-leave="onAfterLeave"
  @leave-cancelled="onLeaveCancelled"
>
  <div v-if="show">Content</div>
</Transition>
```

## Hook Signatures

```typescript
// Enter hooks
function onBeforeEnter(el: HTMLElement): void
function onEnter(el: HTMLElement, done: () => void): void
function onAfterEnter(el: HTMLElement): void
function onEnterCancelled(el: HTMLElement): void

// Leave hooks
function onBeforeLeave(el: HTMLElement): void
function onLeave(el: HTMLElement, done: () => void): void
function onAfterLeave(el: HTMLElement): void
function onLeaveCancelled(el: HTMLElement): void // v-show only
```

## Basic JavaScript Animation

```vue
<script setup>
function onBeforeEnter(el: HTMLElement) {
  el.style.opacity = '0'
  el.style.transform = 'scale(0.9)'
}

function onEnter(el: HTMLElement, done: () => void) {
  // Start animation
  el.offsetHeight // Force reflow
  
  el.style.transition = 'all 0.3s ease'
  el.style.opacity = '1'
  el.style.transform = 'scale(1)'
  
  // Call done when animation completes
  setTimeout(done, 300)
}

function onLeave(el: HTMLElement, done: () => void) {
  el.style.transition = 'all 0.3s ease'
  el.style.opacity = '0'
  el.style.transform = 'scale(0.9)'
  
  setTimeout(done, 300)
}
</script>

<template>
  <Transition
    :css="false"
    @before-enter="onBeforeEnter"
    @enter="onEnter"
    @leave="onLeave"
  >
    <div v-if="show">Content</div>
  </Transition>
</template>
```

## Using GSAP

```vue
<script setup>
import gsap from 'gsap'

function onEnter(el: HTMLElement, done: () => void) {
  gsap.from(el, {
    opacity: 0,
    scale: 0.8,
    y: 20,
    duration: 0.5,
    ease: 'back.out(1.7)',
    onComplete: done
  })
}

function onLeave(el: HTMLElement, done: () => void) {
  gsap.to(el, {
    opacity: 0,
    scale: 0.8,
    y: -20,
    duration: 0.3,
    ease: 'power2.in',
    onComplete: done
  })
}
</script>

<template>
  <Transition :css="false" @enter="onEnter" @leave="onLeave">
    <div v-if="show">Animated with GSAP!</div>
  </Transition>
</template>
```

## Staggered Enter with GSAP

```vue
<script setup>
import gsap from 'gsap'

function onBeforeEnter(el: HTMLElement) {
  gsap.set(el, { opacity: 0, y: 20 })
}

function onEnter(el: HTMLElement, done: () => void) {
  const delay = parseInt(el.dataset.index || '0') * 0.1
  
  gsap.to(el, {
    opacity: 1,
    y: 0,
    duration: 0.4,
    delay,
    ease: 'power2.out',
    onComplete: done
  })
}

function onLeave(el: HTMLElement, done: () => void) {
  gsap.to(el, {
    opacity: 0,
    y: -20,
    duration: 0.3,
    onComplete: done
  })
}
</script>

<template>
  <TransitionGroup
    :css="false"
    @before-enter="onBeforeEnter"
    @enter="onEnter"
    @leave="onLeave"
    tag="ul"
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

## The css="false" Prop

```vue
<!-- Tell Vue to skip CSS detection -->
<Transition :css="false" @enter="onEnter" @leave="onLeave">
  <div v-if="show">...</div>
</Transition>
```

When using JS-only animations, add `:css="false"` to:
- Skip CSS class checks
- Improve performance
- Avoid CSS interference

## Appear Animation

Animate on initial render:

```vue
<Transition
  appear
  @appear="onAppear"
  @after-appear="onAfterAppear"
>
  <div>Animates on mount</div>
</Transition>
```

```vue
<script setup>
import gsap from 'gsap'

function onAppear(el: HTMLElement, done: () => void) {
  gsap.from(el, {
    opacity: 0,
    y: 50,
    duration: 0.8,
    ease: 'power3.out',
    onComplete: done
  })
}
</script>
```

## Resources

- [JavaScript Hooks](https://vuejs.org/guide/built-ins/transition.html#javascript-hooks) — JavaScript animation hooks

---

> 📘 *This lesson is part of the [Vue Animations & Transitions](https://stanza.dev/courses/vue-animations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*