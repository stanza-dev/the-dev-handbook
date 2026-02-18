---
source_course: "svelte-5-migration"
source_lesson: "svelte-5-migration-running-script"
---

# The Automated Migration Tool

Svelte provides a powerful migration script that handles much of the tedious work.

## Running the Migration

```bash
# Navigate to your project
cd my-svelte-app

# Run the migration
npx sv migrate svelte-5
```

## What the Script Does

**1. Updates Configuration Files:**
- `svelte.config.js`
- `vite.config.js`
- `tsconfig.json`

**2. Converts Event Syntax:**
```svelte
<!-- Before -->
<button on:click={handler}>Click</button>

<!-- After -->
<button onclick={handler}>Click</button>
```

**3. Updates Imports:**
```javascript
// Some internal imports may change
```

## What It DOESN'T Do (You Must Do Manually)

- Convert `let x = 0` to `let x = $state(0)`
- Convert `$: derived = ...` to `$derived(...)`
- Replace `createEventDispatcher` with callback props
- Convert `<slot />` to snippets
- Migrate Svelte stores to Runes

## Running on Specific Files

```bash
# Migrate a specific component
npx sv migrate svelte-5 src/lib/MyComponent.svelte

# Migrate a directory
npx sv migrate svelte-5 src/routes/
```

## After Running the Script

1. **Review changes** - Check git diff for what changed
2. **Run your app** - Make sure it still works
3. **Fix errors** - Address any TypeScript or runtime errors
4. **Test thoroughly** - Run your test suite

📖 [sv migrate documentation](https://svelte.dev/docs/cli/sv-migrate)

---

> 📘 *This lesson is part of the [Moving to Svelte 5](https://stanza.dev/courses/svelte-5-migration) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*