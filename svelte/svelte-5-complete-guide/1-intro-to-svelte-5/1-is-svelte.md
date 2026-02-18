---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-what-is-svelte"
---

# What Makes Svelte Different?

When you use frameworks like React or Vue, your application ships with a **runtime** — a large piece of JavaScript that runs in the browser to manage updates to the page. This runtime uses a technique called **Virtual DOM diffing**, which compares the previous and current state of your UI to figure out what changed.

Svelte takes a radically different approach: it's a **compiler**.

## The Compiler Approach

Instead of shipping a runtime, Svelte analyzes your code at **build time** and generates highly optimized vanilla JavaScript. This JavaScript knows exactly which parts of the DOM need to update when your data changes — no diffing required.

Think of it this way:
- **React/Vue**: "Here's a blueprint and a construction crew (runtime). The crew figures out what to build every time something changes."
- **Svelte**: "Here's a finished building with smart sensors (compiled code). It knows exactly which lights to turn on when you flip a switch."

## Why Svelte 5?

Svelte 5 (released in late 2024) introduces the most significant change since the framework's creation: **Runes**.

In previous versions, Svelte used "magic" to make things reactive. You'd declare `let count = 0` and Svelte would *implicitly* track it. While this felt magical, it had limitations — it only worked inside `.svelte` files and could be confusing.

**Runes** are explicit symbols (like `$state`, `$derived`, `$effect`) that clearly mark what is reactive. This brings:

- **Clarity**: You know exactly what's reactive by looking at the code
- **Universality**: Runes work in `.js` and `.ts` files, not just components
- **Type Safety**: Better TypeScript support and IDE autocomplete
- **Predictability**: No more guessing about reactivity

## Key Benefits of Svelte

| Feature | What It Means |
|---------|---------------|
| **Tiny Bundle Size** | No runtime = smaller downloads for users |
| **Lightning Fast** | Direct DOM manipulation beats Virtual DOM |
| **Less Boilerplate** | Write 30-40% less code than React |
| **Built-in Features** | Animations, stores, and transitions out of the box |
| **Great DX** | Excellent error messages and tooling |

## Resources

- [Svelte Overview](https://svelte.dev/docs/svelte/overview) — Official introduction to Svelte and its core concepts.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*