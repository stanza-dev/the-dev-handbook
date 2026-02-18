---
source_course: "svelte-5-migration"
source_lesson: "svelte-5-migration-removing-imports"
---

# Cleaning Up After Migration

Once all components use Runes, remove legacy imports and patterns.

## Lifecycle Imports

```javascript
// ❌ Remove these
import { onMount, onDestroy, beforeUpdate, afterUpdate } from 'svelte';

// ✅ Use $effect instead
$effect(() => {
  // Runs after mount and on updates
  return () => {
    // Cleanup (like onDestroy)
  };
});
```

## Event Dispatcher

```javascript
// ❌ Remove
import { createEventDispatcher } from 'svelte';
const dispatch = createEventDispatcher();

// ✅ Already replaced with callback props
let { onsubmit } = $props();
```

## Store Imports (Maybe)

```javascript
// Consider replacing with $state
import { writable, derived } from 'svelte/store';

// Can become:
// store.svelte.js
export const count = $state({ value: 0 });
export const doubled = $derived(count.value * 2);
```

**Note:** Keep stores if they're used across many files or by third-party libs.

## Tick (Still Useful)

```javascript
// ✅ Still valid and useful
import { tick } from 'svelte';

async function handleClick() {
  count++;
  await tick(); // Wait for DOM update
  inputElement.focus();
}
```

## Search for Legacy Patterns

```bash
# Find files with legacy imports
grep -r "createEventDispatcher\|onMount\|beforeUpdate" src/

# Find legacy reactive statements
grep -r "^\s*\$:" src/

# Find old slot usage
grep -r "<slot" src/
```

📖 [Svelte API reference](https://svelte.dev/docs/svelte/svelte)

---

> 📘 *This lesson is part of the [Moving to Svelte 5](https://stanza.dev/courses/svelte-5-migration) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*