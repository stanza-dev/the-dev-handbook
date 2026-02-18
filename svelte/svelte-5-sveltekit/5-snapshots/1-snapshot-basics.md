---
source_course: "svelte-5-sveltekit"
source_lesson: "svelte-5-sveltekit-snapshot-basics"
---

# The Problem: Lost State on Navigation

When users navigate away and come back, local component state is lost:

```svelte
<script>
  // User types a comment...
  let comment = $state('');
</script>

<!-- User navigates away, comes back -->
<!-- comment is now '' again! -->
<textarea bind:value={comment}></textarea>
```

## The Solution: Snapshots

SvelteKit's snapshot feature preserves and restores state:

```svelte
<!-- +page.svelte -->
<script>
  let comment = $state('');
  let scrollPosition = $state(0);
  
  export const snapshot = {
    capture: () => ({
      comment,
      scrollPosition: window.scrollY
    }),
    restore: (value) => {
      comment = value.comment;
      // Restore scroll after render
      tick().then(() => window.scrollTo(0, value.scrollPosition));
    }
  };
</script>

<textarea bind:value={comment}></textarea>
```

## How Snapshots Work

1. User navigates **away** → `capture()` saves state
2. User clicks **back** → `restore()` receives saved state
3. State is stored in browser history (survives refresh!)

## What to Snapshot

**Good candidates:**
- Form inputs (drafts, unsaved data)
- Scroll position
- Expanded/collapsed sections
- Filter/sort selections NOT in URL
- Tab selections

**Don't snapshot:**
- Data that should be in URL (filters, pagination)
- Authentication state
- Large data (there are size limits)

## Complex Snapshot Example

```svelte
<script>
  import { tick } from 'svelte';
  
  let formData = $state({
    title: '',
    body: '',
    tags: []
  });
  let activeTab = $state('write');
  let expandedSections = $state(new Set());
  
  export const snapshot = {
    capture: () => ({
      formData: { ...formData },
      activeTab,
      expandedSections: [...expandedSections],
      scroll: window.scrollY
    }),
    restore: (value) => {
      formData = value.formData;
      activeTab = value.activeTab;
      expandedSections = new Set(value.expandedSections);
      tick().then(() => window.scrollTo(0, value.scroll));
    }
  };
</script>
```

📖 [Snapshots documentation](https://svelte.dev/docs/kit/snapshots)

---

> 📘 *This lesson is part of the [SvelteKit 2 & Svelte 5: The Perfect Duo](https://stanza.dev/courses/svelte-5-sveltekit) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*