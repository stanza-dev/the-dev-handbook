---
source_course: "react"
source_lesson: "react-useeffect-dependencies"
---

# Dependency Array

The dependency array controls when your effect runs:

| Dependency | Behavior |
|------------|----------|
| `[]` | Run once on mount |
| `[a, b]` | Run when `a` or `b` changes |
| No array | Run after every render |

## Rules

1. **Include all reactive values** used inside the effect
2. **Lint will warn you** if you miss dependencies
3. **Don't lie about dependencies** - it causes bugs!

## Code Examples

**Different dependency array patterns**

```tsx
// Run once on mount
useEffect(() => {
  console.log('Component mounted');
}, []);

// Run when count changes
useEffect(() => {
  document.title = `Count: ${count}`;
}, [count]);

// ❌ Bad: missing dependency
useEffect(() => {
  fetchUser(userId); // userId should be in deps!
}, []);

// ✅ Good: all dependencies listed
useEffect(() => {
  fetchUser(userId);
}, [userId]);
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*