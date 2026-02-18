---
source_course: "svelte-5-state"
source_lesson: "svelte-5-state-prop-drilling"
---

# When Lifting State Breaks Down

Lifting state works great until your component tree gets deep. Then you encounter **prop drilling**.

## What is Prop Drilling?

Passing props through multiple levels of components that don't use them:

```svelte
<!-- App.svelte -->
<script>
  let user = $state({ name: 'Alice', role: 'admin' });
</script>
<Layout {user}>
  <Content />
</Layout>
```

```svelte
<!-- Layout.svelte -->
<script>
  let { user, children } = $props();
  // Layout doesn't use 'user', just passes it down
</script>
<Header {user} />
<main>{@render children()}</main>
<Footer />
```

```svelte
<!-- Header.svelte -->
<script>
  let { user } = $props();
  // Header doesn't use 'user' either!
</script>
<nav>
  <Logo />
  <Navigation />
  <UserMenu {user} /> <!-- Finally used here! -->
</nav>
```

```svelte
<!-- UserMenu.svelte -->
<script>
  let { user } = $props();
  // This is where we actually need user!
</script>
<div>Welcome, {user.name}!</div>
```

## The Problems with Prop Drilling

**1. Verbose and Repetitive**
```
App → Layout → Header → UserMenu
     (pass)   (pass)   (finally use!)
```

**2. Tight Coupling**
Every intermediate component must know about `user` even if they don't use it.

**3. Refactoring Nightmare**
Adding a new prop means updating every component in the chain.

**4. Type Noise**
Every component's props type includes things it doesn't care about.

## Signs You Need a Different Solution

- Props passing through 3+ levels unused
- Many components getting "pass-through" props
- Difficulty refactoring prop shapes
- Components becoming less reusable due to prop requirements

## Solutions Preview

We'll cover these in upcoming lessons:
1. **Context API** - For tree-scoped state
2. **Global stores** - For app-wide state (client-only)
3. **URL state** - For shareable state

📖 [Context documentation](https://svelte.dev/docs/svelte/context)

---

> 📘 *This lesson is part of the [Scalable State Management](https://stanza.dev/courses/svelte-5-state) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*