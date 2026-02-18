---
source_course: "svelte-5-motion"
source_lesson: "svelte-5-motion-local-transitions"
---

# Controlling When Transitions Run

By default, transitions run when ANY ancestor block changes. The `local` modifier gives you more control.

## The Default Behavior (Global)

```svelte
<script>
  let showOuter = $state(true);
  let showInner = $state(true);
</script>

{#if showOuter}
  <div>
    {#if showInner}
      <!-- This transitions when showOuter OR showInner changes -->
      <p transition:fade>Inner content</p>
    {/if}
  </div>
{/if}
```

Problem: When you toggle `showOuter`, the inner element also transitions. Sometimes you only want it to transition when `showInner` changes.

## The Local Modifier

```svelte
{#if showOuter}
  <div>
    {#if showInner}
      <!-- Only transitions when showInner changes -->
      <p transition:fade|local>Inner content</p>
    {/if}
  </div>
{/if}
```

With `|local`, the element:
- **Does** transition when its own condition (`showInner`) changes
- **Does NOT** transition when a parent condition (`showOuter`) changes

## When to Use Local

✅ **Use local when:**
- Nested items shouldn't animate when parent hides
- Tab content shouldn't animate when tab panel hides
- Modal content shouldn't animate when modal closes

❌ **Avoid local when:**
- You want coordinated parent-child animations
- The whole section should animate together

## Example: Tabs

```svelte
{#if showTabs}
  <div class="tabs">
    {#if activeTab === 'home'}
      <!-- Only animates on tab switch, not when tabs disappear -->
      <div transition:fade|local>
        Home content
      </div>
    {:else if activeTab === 'profile'}
      <div transition:fade|local>
        Profile content
      </div>
    {/if}
  </div>
{/if}
```

📖 [Local transitions](https://svelte.dev/docs/svelte/transition#Local-vs-global)

---

> 📘 *This lesson is part of the [Motion & Transitions](https://stanza.dev/courses/svelte-5-motion) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*