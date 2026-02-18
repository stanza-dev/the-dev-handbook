---
source_course: "vue-animations"
source_lesson: "vue-animations-watchers-animation"
---

# Animating State with Watchers

Animate value changes using watchers and tweening libraries.

## Number Tweening

```vue
<script setup>
import { ref, watch } from 'vue'

const actualNumber = ref(0)
const displayNumber = ref(0)

watch(actualNumber, (newValue) => {
  const startValue = displayNumber.value
  const difference = newValue - startValue
  const duration = 500
  const startTime = performance.now()
  
  function animate(currentTime: number) {
    const elapsed = currentTime - startTime
    const progress = Math.min(elapsed / duration, 1)
    
    // Easing function (ease-out)
    const eased = 1 - Math.pow(1 - progress, 3)
    
    displayNumber.value = Math.round(startValue + difference * eased)
    
    if (progress < 1) {
      requestAnimationFrame(animate)
    }
  }
  
  requestAnimationFrame(animate)
})

function increment() {
  actualNumber.value += 100
}
</script>

<template>
  <button @click="increment">Add 100</button>
  <p>{{ displayNumber }}</p>
</template>
```

## Using GSAP for Tweening

```vue
<script setup>
import { ref, reactive, watch } from 'vue'
import gsap from 'gsap'

const number = ref(0)
const tweened = reactive({ value: 0 })

watch(number, (newValue) => {
  gsap.to(tweened, {
    value: newValue,
    duration: 0.5,
    ease: 'power2.out'
  })
})
</script>

<template>
  <input type="number" v-model.number="number" />
  <p>{{ tweened.value.toFixed(0) }}</p>
</template>
```

## Color Transitions

```vue
<script setup>
import { ref, computed, watch } from 'vue'
import gsap from 'gsap'

const colors = ['#ff0000', '#00ff00', '#0000ff', '#ffff00']
const colorIndex = ref(0)
const tweenedColor = ref(colors[0])

const currentColor = computed(() => colors[colorIndex.value])

watch(currentColor, (newColor) => {
  gsap.to(tweenedColor, {
    value: newColor,
    duration: 0.5
  })
})

function nextColor() {
  colorIndex.value = (colorIndex.value + 1) % colors.length
}
</script>

<template>
  <div 
    class="color-box" 
    :style="{ backgroundColor: tweenedColor }"
    @click="nextColor"
  >
    Click me
  </div>
</template>
```

## Progress Bar Animation

```vue
<script setup>
import { ref, watch } from 'vue'

const progress = ref(0)
const animatedProgress = ref(0)

watch(progress, (newValue) => {
  const start = animatedProgress.value
  const change = newValue - start
  const duration = 600
  const startTime = performance.now()
  
  function animate(time: number) {
    const elapsed = time - startTime
    const t = Math.min(elapsed / duration, 1)
    
    // Smooth easing
    const eased = t < 0.5 
      ? 4 * t * t * t 
      : 1 - Math.pow(-2 * t + 2, 3) / 2
    
    animatedProgress.value = start + change * eased
    
    if (t < 1) requestAnimationFrame(animate)
  }
  
  requestAnimationFrame(animate)
})
</script>

<template>
  <div class="progress-bar">
    <div 
      class="progress-fill"
      :style="{ width: animatedProgress + '%' }"
    />
  </div>
  <input type="range" v-model.number="progress" min="0" max="100" />
</template>

<style>
.progress-bar {
  width: 100%;
  height: 20px;
  background: #eee;
  border-radius: 10px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #42b883, #35495e);
  transition: none; /* JavaScript handles animation */
}
</style>
```

## Chart Data Animation

```vue
<script setup>
import { ref, reactive, watch } from 'vue'
import gsap from 'gsap'

const data = ref([25, 50, 75, 100, 60])
const tweenedData = reactive([25, 50, 75, 100, 60])

watch(data, (newData) => {
  newData.forEach((value, index) => {
    gsap.to(tweenedData, {
      [index]: value,
      duration: 0.5,
      ease: 'power2.out'
    })
  })
}, { deep: true })

function randomize() {
  data.value = data.value.map(() => Math.floor(Math.random() * 100))
}
</script>

<template>
  <button @click="randomize">Randomize</button>
  
  <div class="chart">
    <div 
      v-for="(value, index) in tweenedData"
      :key="index"
      class="bar"
      :style="{ height: value + '%' }"
    />
  </div>
</template>

<style>
.chart {
  display: flex;
  align-items: flex-end;
  height: 200px;
  gap: 10px;
}

.bar {
  flex: 1;
  background: #42b883;
  border-radius: 4px 4px 0 0;
}
</style>
```

## Resources

- [State-Driven Animations](https://vuejs.org/guide/extras/animation.html#state-driven-animations) — Animating state changes

---

> 📘 *This lesson is part of the [Vue Animations & Transitions](https://stanza.dev/courses/vue-animations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*