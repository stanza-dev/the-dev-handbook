---
source_course: "vue-reactivity-mastery"
source_lesson: "vue-reactivity-mastery-how-reactivity-works"
---

# How Vue's Reactivity System Works

Vue's reactivity system is the magic that makes your UI update automatically when data changes. Understanding how it works will help you write more efficient code and debug issues.

## The Core Concept: Proxy-Based Reactivity

Vue 3 uses JavaScript's `Proxy` object to intercept property access and mutations. Here's a simplified view:

```javascript
// Simplified reactive() implementation
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key) {
      track(target, key)    // Record this dependency
      return target[key]
    },
    set(target, key, value) {
      target[key] = value
      trigger(target, key)  // Notify subscribers
      return true
    }
  })
}
```

When you access a property, Vue **tracks** it as a dependency. When you modify it, Vue **triggers** updates to anything that depends on it.

## Dependency Tracking

Vue automatically tracks which components and computed properties depend on which reactive data:

```
┌─────────────────────────────────────────────┐
│              Reactive State                  │
│         const count = ref(0)                 │
└─────────────────────────────────────────────┘
                    │
         ┌──────────┼──────────┐
         │          │          │
         ▼          ▼          ▼
    ┌─────────┐ ┌─────────┐ ┌─────────┐
    │Component│ │Computed │ │ Watcher │
    │Template │ │Property │ │Callback │
    └─────────┘ └─────────┘ └─────────┘

    When count.value changes, ALL subscribers update!
```

## ref() Under the Hood

`ref()` wraps a value in an object with a reactive `.value` property:

```javascript
// Simplified ref() implementation
function ref(value) {
  const refObject = {
    get value() {
      track(refObject, 'value')  // Track when read
      return value
    },
    set value(newValue) {
      value = newValue
      trigger(refObject, 'value')  // Trigger when written
    }
  }
  return refObject
}
```

This is why you need `.value` in JavaScript—it's the property that Vue intercepts!

## reactive() Under the Hood

`reactive()` returns a Proxy that intercepts ALL property access:

```javascript
const state = reactive({ count: 0, name: 'Vue' })

// Every property access is tracked
console.log(state.count)  // track() called
console.log(state.name)   // track() called

// Every mutation triggers updates
state.count++             // trigger() called
state.name = 'Vue 3'      // trigger() called
```

## Why Templates Don't Need .value

Vue's template compiler automatically adds `.value` for refs:

```vue
<script setup>
const count = ref(0)  // ref
</script>

<template>
  <!-- Vue compiles this to: count.value -->
  <p>{{ count }}</p>
</template>
```

## Reactivity Caveats

### 1. Destructuring Breaks Reactivity

```javascript
const state = reactive({ count: 0 })

// ❌ count is now a plain number, not reactive
let { count } = state
count++  // Doesn't trigger updates!

// ✅ Keep using state.count
state.count++

// ✅ Or use toRefs()
const { count } = toRefs(state)
count.value++  // This works!
```

### 2. Replacing Reactive Objects

```javascript
let state = reactive({ count: 0 })

// ❌ Loses reactivity - new object isn't tracked
state = reactive({ count: 1 })

// ✅ Modify properties instead
state.count = 1

// ✅ Or use ref for the container
const state = ref({ count: 0 })
state.value = { count: 1 }  // This works!
```

### 3. Reactivity Only Works on Objects

```javascript
// ❌ Primitives can't be made reactive directly
const count = reactive(0)  // Returns 0, not a proxy

// ✅ Use ref() for primitives
const count = ref(0)
```

## The Effect System

Under the hood, Vue uses an "effect" system:

```
┌──────────────────────────────────────────────────────┐
│                    Running Effect                     │
│  (template render, computed getter, watcher callback) │
└──────────────────────────────────────────────────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │ Accesses state.count │
              └─────────────────────┘
                         │
                         ▼
           ┌───────────────────────────┐
           │ Vue records: this effect  │
           │ depends on state.count    │
           └───────────────────────────┘
                         │
          When state.count changes:
                         │
                         ▼
           ┌───────────────────────────┐
           │ Vue re-runs all effects   │
           │ that depend on it         │
           └───────────────────────────┘
```

This automatic tracking is what makes Vue's reactivity feel "magical"!

## Resources

- [Reactivity in Depth](https://vuejs.org/guide/extras/reactivity-in-depth.html) — Official deep dive into Vue's reactivity system

---

> 📘 *This lesson is part of the [Vue Reactivity Mastery](https://stanza.dev/courses/vue-reactivity-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*