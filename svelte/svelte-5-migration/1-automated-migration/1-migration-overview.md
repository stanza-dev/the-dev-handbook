---
source_course: "svelte-5-migration"
source_lesson: "svelte-5-migration-migration-overview"
---

# Planning Your Svelte 5 Migration

Migrating to Svelte 5 can be done incrementally. Here's an overview of the process.

## Migration Phases

```
┌─────────────────────────────────────────────────────────┐
│                  PHASE 1: Preparation                   │
│  • Update dependencies (svelte@5, sveltekit@2)         │
│  • Run the migration script                             │
│  • Fix any breaking changes                             │
└───────────────────────────┬─────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                  PHASE 2: Get Running                   │
│  • App works in "legacy mode"                          │
│  • Old syntax still supported                          │
│  • No Runes yet                                         │
└───────────────────────────┬─────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│               PHASE 3: Incremental Adoption             │
│  • Migrate components one by one to Runes              │
│  • Convert events, slots, stores                        │
│  • Test thoroughly                                      │
└───────────────────────────┬─────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                  PHASE 4: Full Runes                    │
│  • All components use Runes                             │
│  • Legacy imports removed                               │
│  • Modern Svelte 5 codebase                            │
└─────────────────────────────────────────────────────────┘
```

## What Changes in Svelte 5?

| Old (Svelte 4) | New (Svelte 5) |
|---------------|----------------|
| `let count = 0` (magic reactivity) | `let count = $state(0)` |
| `$: doubled = count * 2` | `let doubled = $derived(count * 2)` |
| `$: { console.log(count) }` | `$effect(() => { console.log(count) })` |
| `export let name` | `let { name } = $props()` |
| `on:click` | `onclick` |
| `<slot />` | `{@render children()}` |
| `createEventDispatcher()` | Callback props |
| `$$restProps` | `let { ...rest } = $props()` |

## Good News: Backward Compatibility

Svelte 5 supports "legacy mode" - your Svelte 4 code still works! You can migrate gradually.

📖 [Migration guide](https://svelte.dev/docs/svelte/v5-migration-guide)

---

> 📘 *This lesson is part of the [Moving to Svelte 5](https://stanza.dev/courses/svelte-5-migration) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*