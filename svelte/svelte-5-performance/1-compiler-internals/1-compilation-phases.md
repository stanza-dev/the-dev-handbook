---
source_course: "svelte-5-performance"
source_lesson: "svelte-5-performance-compilation-phases"
---

# Understanding Svelte's Compilation Pipeline

Unlike runtime frameworks like React or Vue, Svelte does most of its work at build time. This is what makes it so fast.

## The Compilation Process

```
┌─────────────────────────────────────────────────────────┐
│                    YOUR CODE                            │
│  ┌─────────────┐  ┌───────────┐  ┌─────────────────┐   │
│  │  Template   │  │   Script  │  │     Styles      │   │
│  │  (<div>)    │  │ ($state)  │  │   (.class {})   │   │
│  └──────┬──────┘  └─────┬─────┘  └────────┬────────┘   │
└─────────┼───────────────┼─────────────────┼────────────┘
          │               │                 │
          ▼               ▼                 ▼
┌─────────────────────────────────────────────────────────┐
│                    PHASE 1: PARSING                     │
│         Convert source code to AST (Abstract Syntax    │
│         Tree) - a data structure representing code      │
└───────────────────────────┬─────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                    PHASE 2: ANALYSIS                    │
│         • Find reactive statements ($state, $derived)   │
│         • Track dependencies between values             │
│         • Identify which DOM nodes need updates         │
└───────────────────────────┬─────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                    PHASE 3: CODE GENERATION             │
│         Generate optimized JavaScript that:             │
│         • Creates DOM nodes directly                    │
│         • Updates only what changed                     │
│         • Scopes CSS automatically                      │
└───────────────────────────┬─────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                    OUTPUT                               │
│  • Vanilla JavaScript (no framework runtime needed)    │
│  • Tiny bundle size                                     │
│  • Direct DOM manipulation                              │
└─────────────────────────────────────────────────────────┘
```

## Why This Matters

**React approach:**
- Ships entire React runtime (~40KB gzipped)
- Runtime diffing of Virtual DOM on every update
- Component functions re-run on every state change

**Svelte approach:**
- No framework runtime shipped (just your code + tiny helpers)
- Compiler knows exactly which DOM nodes to update
- Only the affected parts run, not entire components

📖 [Svelte compiler overview](https://svelte.dev/docs/svelte/overview)

---

> 📘 *This lesson is part of the [Under the Hood & Performance](https://stanza.dev/courses/svelte-5-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*