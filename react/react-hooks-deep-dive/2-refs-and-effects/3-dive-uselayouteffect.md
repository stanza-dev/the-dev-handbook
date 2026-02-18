---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-uselayouteffect"
---

# useLayoutEffect: Before the Browser Paints

`useLayoutEffect` is identical to `useEffect`, but it fires **synchronously** after all DOM mutations and **before** the browser paints.

## When to Use It

Use `useLayoutEffect` when you need to:

1. **Measure DOM elements** before the user sees them
2. **Prevent visual flicker** by making synchronous updates
3. **Position elements** based on other elements' dimensions

## Example: Tooltip Positioning

```tsx
function Tooltip({ targetRef, children }: TooltipProps) {
  const tooltipRef = useRef<HTMLDivElement>(null);
  const [position, setPosition] = useState({ top: 0, left: 0 });

  useLayoutEffect(() => {
    if (!targetRef.current || !tooltipRef.current) return;

    const targetRect = targetRef.current.getBoundingClientRect();
    const tooltipRect = tooltipRef.current.getBoundingClientRect();

    setPosition({
      top: targetRect.bottom + 8,
      left: targetRect.left + (targetRect.width - tooltipRect.width) / 2
    });
  }, [targetRef]);

  return (
    <div
      ref={tooltipRef}
      style={{
        position: 'fixed',
        top: position.top,
        left: position.left
      }}
    >
      {children}
    </div>
  );
}
```

## useEffect vs useLayoutEffect

```
Render → DOM Update → useLayoutEffect → Browser Paint → useEffect
```

| Aspect | useEffect | useLayoutEffect |
|--------|-----------|------------------|
| Timing | After paint | Before paint |
| Blocking | Non-blocking | Blocks visual updates |
| Use case | Most effects | DOM measurements |
| SSR | Safe | Warns on server |

## SSR Considerations

`useLayoutEffect` doesn't run on the server, which can cause hydration mismatches. For SSR-safe code:

```tsx
import { useEffect, useLayoutEffect } from 'react';

// Use this instead of useLayoutEffect for SSR compatibility
const useIsomorphicLayoutEffect =
  typeof window !== 'undefined' ? useLayoutEffect : useEffect;
```

## Performance Warning

⚠️ `useLayoutEffect` blocks the browser from painting. Use it sparingly:

```tsx
// 🔴 Bad - expensive computation blocks paint
useLayoutEffect(() => {
  const result = expensiveCalculation();
  setData(result);
}, []);

// ✅ Good - only DOM measurements
useLayoutEffect(() => {
  const rect = ref.current.getBoundingClientRect();
  setDimensions(rect);
}, []);
```

## Resources

- [useLayoutEffect API Reference](https://react.dev/reference/react/useLayoutEffect) — Official React documentation for useLayoutEffect hook

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*