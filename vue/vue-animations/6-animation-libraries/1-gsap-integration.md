---
source_course: "vue-animations"
source_lesson: "vue-animations-gsap-integration"
---

# Integrating GSAP with Vue

GSAP (GreenSock Animation Platform) is a powerful animation library that works seamlessly with Vue.

## Installation

```bash
npm install gsap
```

## Basic GSAP Animation

```vue
<script setup>
import { ref, onMounted } from 'vue'
import gsap from 'gsap'

const boxRef = ref<HTMLElement | null>(null)

onMounted(() => {
  if (boxRef.value) {
    gsap.to(boxRef.value, {
      x: 200,
      rotation: 360,
      duration: 1,
      ease: 'power2.out'
    })
  }
})
</script>

<template>
  <div ref="boxRef" class="box">Animated!</div>
</template>
```

## GSAP with Transition Hooks

```vue
<script setup>
import { ref } from 'vue'
import gsap from 'gsap'

const show = ref(true)

function onEnter(el: HTMLElement, done: () => void) {
  gsap.from(el, {
    opacity: 0,
    y: -50,
    scale: 0.8,
    duration: 0.5,
    ease: 'back.out(1.7)',
    onComplete: done
  })
}

function onLeave(el: HTMLElement, done: () => void) {
  gsap.to(el, {
    opacity: 0,
    y: 50,
    scale: 0.8,
    duration: 0.3,
    ease: 'power2.in',
    onComplete: done
  })
}
</script>

<template>
  <button @click="show = !show">Toggle</button>
  
  <Transition
    :css="false"
    @enter="onEnter"
    @leave="onLeave"
  >
    <div v-if="show" class="card">GSAP Animated Card</div>
  </Transition>
</template>
```

## Staggered Animations with GSAP

```vue
<script setup>
import { ref } from 'vue'
import gsap from 'gsap'

const items = ref([1, 2, 3, 4, 5])

function onBeforeEnter(el: HTMLElement) {
  el.style.opacity = '0'
  el.style.transform = 'translateY(30px)'
}

function onEnter(el: HTMLElement, done: () => void) {
  const index = Number(el.dataset.index)
  gsap.to(el, {
    opacity: 1,
    y: 0,
    duration: 0.4,
    delay: index * 0.1,
    ease: 'power2.out',
    onComplete: done
  })
}

function onLeave(el: HTMLElement, done: () => void) {
  const index = Number(el.dataset.index)
  gsap.to(el, {
    opacity: 0,
    y: -30,
    duration: 0.3,
    delay: index * 0.05,
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
  >
    <div
      v-for="(item, index) in items"
      :key="item"
      :data-index="index"
      class="item"
    >
      Item {{ item }}
    </div>
  </TransitionGroup>
</template>
```

## GSAP Timeline

```vue
<script setup>
import { ref, onMounted } from 'vue'
import gsap from 'gsap'

const containerRef = ref<HTMLElement | null>(null)

onMounted(() => {
  const tl = gsap.timeline({ defaults: { duration: 0.5 } })
  
  tl.from('.header', { y: -50, opacity: 0 })
    .from('.content', { x: -100, opacity: 0 }, '-=0.2')
    .from('.button', { scale: 0, ease: 'back.out(2)' })
    .from('.footer', { y: 50, opacity: 0 }, '-=0.3')
})
</script>
```

## Reactive Animations with watchEffect

```vue
<script setup>
import { ref, watchEffect } from 'vue'
import gsap from 'gsap'

const progress = ref(0)
const progressBar = ref<HTMLElement | null>(null)

watchEffect(() => {
  if (progressBar.value) {
    gsap.to(progressBar.value, {
      width: `${progress.value}%`,
      duration: 0.5,
      ease: 'power2.out'
    })
  }
})
</script>

<template>
  <input type="range" v-model.number="progress" min="0" max="100" />
  <div class="progress-container">
    <div ref="progressBar" class="progress-bar"></div>
  </div>
</template>
```

## Resources

- [GSAP Documentation](https://greensock.com/docs/) — Official GSAP documentation

---

> 📘 *This lesson is part of the [Vue Animations & Transitions](https://stanza.dev/courses/vue-animations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*