---
source_course: "vue-reactivity-mastery"
source_lesson: "vue-reactivity-mastery-effect-scope"
---

# Managing Effects with effectScope

`effectScope()` creates a scope that can capture all reactive effects within it, allowing them to be disposed together.

## Basic Usage

```typescript
import { effectScope, ref, computed, watch, watchEffect } from 'vue'

const scope = effectScope()

scope.run(() => {
  const count = ref(0)
  const double = computed(() => count.value * 2)
  
  watch(count, () => console.log('count changed'))
  watchEffect(() => console.log('double is', double.value))
})

// Later, dispose ALL effects at once
scope.stop()
```

## Why Use effectScope?

Without scopes, you'd need to track each effect manually:

```typescript
// ❌ Manual tracking is tedious
const stopWatch1 = watch(/* ... */)
const stopWatch2 = watch(/* ... */)
const stopEffect = watchEffect(/* ... */)

function cleanup() {
  stopWatch1()
  stopWatch2()
  stopEffect()
  // Easy to miss one!
}

// ✅ With scope, cleanup is automatic
const scope = effectScope()

scope.run(() => {
  watch(/* ... */)
  watch(/* ... */)
  watchEffect(/* ... */)
})

function cleanup() {
  scope.stop()  // Stops everything
}
```

## Building Reusable Composables

```typescript
import { effectScope, onScopeDispose } from 'vue'

function useMouseTracker() {
  const scope = effectScope()
  
  return scope.run(() => {
    const x = ref(0)
    const y = ref(0)
    
    function update(event: MouseEvent) {
      x.value = event.clientX
      y.value = event.clientY
    }
    
    window.addEventListener('mousemove', update)
    
    // Cleanup when scope is stopped
    onScopeDispose(() => {
      window.removeEventListener('mousemove', update)
    })
    
    return { x, y }
  })!
}

// Usage in component
const { x, y } = useMouseTracker()
// Automatically cleaned up when component unmounts
```

## getCurrentScope and onScopeDispose

```typescript
import { 
  getCurrentScope, 
  onScopeDispose, 
  effectScope,
  watchEffect 
} from 'vue'

function useEventListener(
  target: EventTarget,
  event: string,
  callback: EventListener
) {
  // Check if we're inside a scope
  if (getCurrentScope()) {
    target.addEventListener(event, callback)
    
    // Will be called when the current scope stops
    onScopeDispose(() => {
      target.removeEventListener(event, callback)
    })
  } else {
    console.warn('useEventListener called outside of scope')
  }
}

// Usage
const scope = effectScope()

scope.run(() => {
  useEventListener(window, 'resize', handleResize)
  useEventListener(document, 'click', handleClick)
})

// Both listeners removed
scope.stop()
```

## Nested Scopes

```typescript
import { effectScope } from 'vue'

const parentScope = effectScope()

parentScope.run(() => {
  const childScope = effectScope()
  
  childScope.run(() => {
    // Child effects
  })
  
  // Child scope is automatically stopped when parent stops
})

parentScope.stop()  // Also stops childScope
```

### Detached Scopes

```typescript
import { effectScope } from 'vue'

const parentScope = effectScope()

parentScope.run(() => {
  // Detached: not stopped when parent stops
  const detachedScope = effectScope(true)
  
  detachedScope.run(() => {
    // These effects survive parent.stop()
  })
})

parentScope.stop()  // detachedScope still runs
```

## Practical Example: Feature Toggle

```typescript
import { effectScope, ref, computed, watch } from 'vue'

function useFeature(enabled: Ref<boolean>) {
  let scope: EffectScope | null = null
  const data = ref<Data | null>(null)
  
  watch(
    enabled,
    (isEnabled) => {
      if (isEnabled) {
        // Start feature
        scope = effectScope()
        scope.run(() => {
          // Set up all feature effects
          const interval = setInterval(fetchData, 5000)
          watchEffect(() => processData(data.value))
          
          onScopeDispose(() => {
            clearInterval(interval)
          })
        })
      } else {
        // Stop feature
        scope?.stop()
        scope = null
        data.value = null
      }
    },
    { immediate: true }
  )
  
  return { data }
}
```

## In VueUse

The VueUse library extensively uses `effectScope` internally:

```typescript
// From VueUse: simplified createSharedComposable
import { effectScope } from 'vue'

function createSharedComposable<T>(composable: () => T): () => T {
  let subscribers = 0
  let state: T | null = null
  let scope: EffectScope | null = null
  
  return () => {
    subscribers++
    
    if (!scope) {
      scope = effectScope(true)
      state = scope.run(composable)!
    }
    
    onScopeDispose(() => {
      subscribers--
      if (subscribers === 0) {
        scope?.stop()
        scope = null
        state = null
      }
    })
    
    return state!
  }
}
```

## Resources

- [effectScope](https://vuejs.org/api/reactivity-advanced.html#effectscope) — Official API documentation for effectScope

---

> 📘 *This lesson is part of the [Vue Reactivity Mastery](https://stanza.dev/courses/vue-reactivity-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*