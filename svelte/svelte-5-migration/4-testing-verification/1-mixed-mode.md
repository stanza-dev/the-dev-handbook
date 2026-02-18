---
source_course: "svelte-5-migration"
source_lesson: "svelte-5-migration-mixed-mode"
---

# Incremental Migration with Mixed Mode

You don't have to migrate everything at once! Svelte 5 supports running legacy and runes components side by side.

## How It Works

```
Your App
├── LegacyComponent.svelte  ← Uses Svelte 4 syntax
├── NewComponent.svelte     ← Uses Runes
└── AnotherLegacy.svelte    ← Uses Svelte 4 syntax
```

All three work together seamlessly!

## The Per-Component Rule

A single component must be ALL legacy OR ALL runes:

```svelte
<!-- ❌ DOESN'T WORK - Mixed mode in one file -->
<script>
  let count = 0;        // Legacy reactive
  $effect(() => {       // Rune
    console.log(count); // Oops! count isn't reactive now
  });
</script>
```

Once you use ONE rune, the component switches to "runes mode":
- `let x = 0` is no longer reactive
- `$:` statements don't work
- You must use `$state`, `$derived`, etc.

```svelte
<!-- ✅ All Runes -->
<script>
  let count = $state(0);
  $effect(() => {
    console.log(count); // Works!
  });
</script>
```

## Migration Strategy

**Option 1: Bottom-up**
Start with leaf components (no children), work up.

**Option 2: Top-down**
Start with root/layout, work down.

**Option 3: Feature-by-feature**
Migrate complete features at once.

## Checking Component Mode

Svelte 5 logs a warning if you mix syntax in one component during development.

📖 [Migration guide](https://svelte.dev/docs/svelte/v5-migration-guide)

---

> 📘 *This lesson is part of the [Moving to Svelte 5](https://stanza.dev/courses/svelte-5-migration) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*