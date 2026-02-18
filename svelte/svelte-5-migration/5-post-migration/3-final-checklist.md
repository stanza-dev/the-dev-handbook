---
source_course: "svelte-5-migration"
source_lesson: "svelte-5-migration-final-checklist"
---

# Final Migration Checklist

Before considering your migration complete, verify these items.

## ✅ Code Changes

- [ ] All `let x = value` → `let x = $state(value)`
- [ ] All `$: derived = ...` → `let derived = $derived(...)`
- [ ] All `$: { sideEffect }` → `$effect(() => { ... })`
- [ ] All `export let` → `let { ... } = $props()`
- [ ] All `on:event` → `onevent`
- [ ] All `<slot />` → `{@render children()}`
- [ ] All `createEventDispatcher` → callback props
- [ ] All `$$props`/`$$restProps` → destructured `$props()`

## ✅ Imports Cleaned

- [ ] Removed unused `onMount`, `onDestroy`
- [ ] Removed `beforeUpdate`, `afterUpdate`
- [ ] Removed `createEventDispatcher`
- [ ] Evaluated store usage (keep or convert)

## ✅ Testing

- [ ] All unit tests pass
- [ ] All integration tests pass
- [ ] All E2E tests pass
- [ ] Manual QA complete
- [ ] No console warnings about mixed mode

## ✅ TypeScript (if applicable)

- [ ] Props have proper types
- [ ] State has proper types
- [ ] No `any` types introduced
- [ ] Build passes with strict mode

## ✅ Performance

- [ ] No unnecessary effects
- [ ] Derived values used for expensive calculations
- [ ] Effect cleanup in place
- [ ] No memory leaks in dev tools

## ✅ Documentation

- [ ] README updated if needed
- [ ] Component docs updated
- [ ] Breaking changes documented for library consumers

## Final Steps

```bash
# Clean build
rm -rf .svelte-kit node_modules/.vite
npm install
npm run build

# Run full test suite
npm run test

# Check for any warnings
npm run dev 2>&1 | grep -i warn
```

🎉 **Congratulations on completing your Svelte 5 migration!**

📖 [Migration guide](https://svelte.dev/docs/svelte/v5-migration-guide)

---

> 📘 *This lesson is part of the [Moving to Svelte 5](https://stanza.dev/courses/svelte-5-migration) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*