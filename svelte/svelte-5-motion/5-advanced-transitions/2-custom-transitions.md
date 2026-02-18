---
source_course: "svelte-5-motion"
source_lesson: "svelte-5-motion-custom-transitions"
---

# Build Your Own Transitions

Svelte's built-in transitions are just functions. You can create your own!

## Transition Function Signature

```typescript
function myTransition(node: HTMLElement, params: any): {
  delay?: number;
  duration?: number;
  easing?: (t: number) => number;
  css?: (t: number, u: number) => string;
  tick?: (t: number, u: number) => void;
}
```

## CSS-Based Custom Transition

```javascript
function typewriter(node, { speed = 1 }) {
  const text = node.textContent;
  const duration = text.length / (speed * 0.01);
  
  return {
    duration,
    tick: (t) => {
      const i = Math.trunc(text.length * t);
      node.textContent = text.slice(0, i);
    }
  };
}
```

```svelte
{#if visible}
  <p transition:typewriter={{ speed: 2 }}>
    This text types out letter by letter!
  </p>
{/if}
```

## Understanding t and u

- `t` goes from 0 to 1 during intro (entering)
- `t` goes from 1 to 0 during outro (leaving)
- `u` is always `1 - t`

```javascript
function spin(node, { duration = 400 }) {
  return {
    duration,
    css: (t) => `
      transform: rotate(${t * 360}deg);
      opacity: ${t};
    `
  };
}
```

## CSS vs Tick

**Use `css`** for transforms, opacity, colors:
- GPU accelerated
- Runs off main thread
- More performant

**Use `tick`** for:
- Text content changes
- Canvas animations
- Things CSS can't do

## Advanced: Combining Techniques

```javascript
function whoosh(node, { duration = 400, direction = 'left' }) {
  const style = getComputedStyle(node);
  const transform = style.transform === 'none' ? '' : style.transform;
  
  const x = direction === 'left' ? -100 : 100;
  
  return {
    duration,
    easing: quintOut,
    css: (t, u) => `
      transform: ${transform} translateX(${u * x}%) rotate(${u * 10}deg);
      opacity: ${t};
      filter: blur(${u * 4}px);
    `
  };
}
```

📖 [Custom transitions](https://svelte.dev/docs/svelte/transition#Custom-transition-functions)

---

> 📘 *This lesson is part of the [Motion & Transitions](https://stanza.dev/courses/svelte-5-motion) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*