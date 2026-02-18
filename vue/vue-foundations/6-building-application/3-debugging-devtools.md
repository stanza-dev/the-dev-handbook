---
source_course: "vue-foundations"
source_lesson: "vue-foundations-debugging-devtools"
---

# Debugging with Vue DevTools

Vue DevTools is an essential browser extension for developing Vue applications. It provides powerful inspection and debugging capabilities.

## Installing Vue DevTools

1. **Chrome**: Search "Vue.js devtools" in Chrome Web Store
2. **Firefox**: Search "Vue.js devtools" in Firefox Add-ons
3. **Edge**: Search "Vue.js devtools" in Microsoft Edge Add-ons

After installation, you'll see a new "Vue" tab in your browser's developer tools.

## Component Inspector

The component inspector lets you explore your component tree:

### Viewing Component Hierarchy

- See all components in a tree structure
- Click any component to inspect it
- View parent-child relationships

### Inspecting Component State

For each component, you can see:

- **Props**: Values passed from parent
- **State** (refs/reactive): Component's reactive data
- **Computed**: Computed property values
- **Provide/Inject**: Provided and injected values

### Live Editing

You can edit state directly in DevTools:

1. Select a component
2. Find the value you want to change
3. Click and edit the value
4. See your UI update in real-time!

## Timeline

The timeline shows events over time:

- **Component events**: Mount, update, unmount
- **User events**: Clicks, inputs, etc.
- **Performance**: Rendering time

## Useful Debugging Tips

### Console Inspection

Access the selected component in the console:

```javascript
// After selecting a component in DevTools
$vm  // The component instance
$vm.count  // Access refs (auto-unwrapped)
$vm.props  // Access props
```

### Adding Debug Points

```vue
<script setup>
import { ref, watch } from 'vue'

const count = ref(0)

// Debug with watch
watch(count, (newVal, oldVal) => {
  console.log(`count changed: ${oldVal} → ${newVal}`)
  debugger  // Pause execution here
})
</script>
```

### Template Debugging

```vue
<template>
  <!-- Quick debug output -->
  <pre>{{ JSON.stringify(someData, null, 2) }}</pre>
  
  <!-- Conditional debug -->
  <div v-if="false">
    Debug: {{ complexValue }}
  </div>
</template>
```

### Console Utilities

```vue
<script setup>
import { ref, onMounted } from 'vue'

const items = ref([])

onMounted(async () => {
  console.time('fetch items')
  const response = await fetch('/api/items')
  items.value = await response.json()
  console.timeEnd('fetch items')  // Shows: "fetch items: 234ms"
  
  console.table(items.value)  // Nice table format
  console.group('Item details')
  items.value.forEach(item => console.log(item))
  console.groupEnd()
})
</script>
```

## Common Issues and Solutions

### "Reactivity Not Working"

```vue
<script setup>
import { ref, reactive } from 'vue'

// ❌ Common mistake: losing reactivity
const state = reactive({ count: 0 })
let { count } = state  // count is now a plain number!
count++  // This won't update the UI

// ✅ Solution: use toRefs or access directly
state.count++  // This works

// Or
import { toRefs } from 'vue'
const { count } = toRefs(state)
count.value++  // This works
</script>
```

### "Component Not Updating"

```vue
<script setup>
import { ref } from 'vue'

const items = ref([{ id: 1, name: 'Item' }])

// ❌ Mutating object property doesn't trigger re-render in some cases
items.value[0].name = 'Updated'  // Might not work as expected

// ✅ Ensure you're triggering reactivity
items.value = [...items.value]  // Create new array reference
// Or use Vue's array methods
items.value.splice(0, 1, { ...items.value[0], name: 'Updated' })
</script>
```

### "Event Not Firing"

```vue
<!-- ❌ Event name mismatch -->
<ChildComponent @updateValue="handler" />  <!-- kebab-case -->

<!-- In ChildComponent -->
emit('update-value')  <!-- Different casing! -->

<!-- ✅ Be consistent -->
<ChildComponent @update-value="handler" />
emit('update-value')  <!-- Match! -->
```

## Performance Profiling

Vue DevTools includes a performance tab:

1. Click "Timeline" in Vue DevTools
2. Enable recording
3. Interact with your app
4. Stop recording
5. Analyze component render times

Look for:
- Components that re-render unnecessarily
- Slow computed properties
- Expensive watchers

## Best Practices

1. **Keep DevTools open** during development
2. **Use descriptive component names** for easier debugging
3. **Log meaningful messages** with context
4. **Use the timeline** to understand event sequences
5. **Profile periodically** to catch performance issues early

## Resources

- [Vue DevTools](https://devtools.vuejs.org/) — Official Vue DevTools documentation

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*