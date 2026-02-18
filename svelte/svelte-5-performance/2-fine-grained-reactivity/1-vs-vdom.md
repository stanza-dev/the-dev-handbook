---
source_course: "svelte-5-performance"
source_lesson: "svelte-5-performance-signals-vs-vdom"
---

# Two Philosophies of Reactivity

Let's compare how React and Svelte handle updates.

## React's Virtual DOM Approach

```
1. State changes (setState)
         │
         ▼
2. Entire component function re-runs
         │
         ▼
3. New Virtual DOM tree created
         │
         ▼
4. Diff new tree vs old tree
         │
         ▼
5. Calculate minimal DOM operations
         │
         ▼
6. Apply patches to real DOM
```

**Cost:** O(n) where n = number of elements in component

## Svelte's Signal Approach

```
1. State changes (count++)
         │
         ▼
2. Signal notifies direct subscribers
         │
         ▼
3. Only that DOM node updates
```

**Cost:** O(1) - constant time, regardless of component size

## Visual Comparison

```
┌─────────────────────────────────────────────────────────┐
│              REACT (Virtual DOM)                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │ Component                                        │  │
│  │  ┌────────────────────────────────────────────┐  │  │
│  │  │ State: count = 5                           │  │  │
│  │  │                                            │  │  │
│  │  │ <div>                  ← re-evaluated      │  │  │
│  │  │   <header>...</header> ← re-evaluated      │  │  │
│  │  │   <p>Count: {count}</p>← re-evaluated      │  │  │
│  │  │   <footer>...</footer> ← re-evaluated      │  │  │
│  │  │ </div>                 ← re-evaluated      │  │  │
│  │  └────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────┘  │
│                                                         │
│  Even if only count changed, entire component runs!    │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│              SVELTE (Signals)                           │
│  ┌──────────────────────────────────────────────────┐  │
│  │ Component                                        │  │
│  │  ┌────────────────────────────────────────────┐  │  │
│  │  │ Signal: count = 5                          │  │  │
│  │  │            │                               │  │  │
│  │  │ <div>      │                               │  │  │
│  │  │   <header> │  (untouched)                  │  │  │
│  │  │   <p>Count:└─▶ {count} ← ONLY this updates│  │  │
│  │  │   <footer>    (untouched)                  │  │  │
│  │  │ </div>                                     │  │  │
│  │  └────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────┘  │
│                                                         │
│  Signal has direct link to subscriber - surgical update!│
└─────────────────────────────────────────────────────────┘
```

## Real-World Impact

A component with:
- 1000 static elements
- 1 dynamic number

**React:** Re-renders all 1000 elements, diffs them, updates 1
**Svelte:** Updates only the 1 text node

📖 [Reactivity documentation](https://svelte.dev/docs/svelte/what-are-runes)

---

> 📘 *This lesson is part of the [Under the Hood & Performance](https://stanza.dev/courses/svelte-5-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*