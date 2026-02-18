---
source_course: "react-animation"
source_lesson: "react-animation-enter-exit-animations"
---

# Enter and Exit Animations

Handling animations when components mount/unmount.

## The Challenge

React immediately removes elements from DOM on unmount:

```jsx
// Element disappears instantly - no exit animation!
{isVisible && <div className="fade-in">Content</div>}
```

## Using State for Exit Animation

```jsx
function AnimatedContent({ isVisible, children }) {
  const [shouldRender, setShouldRender] = useState(isVisible);
  const [animationClass, setAnimationClass] = useState('');

  useEffect(() => {
    if (isVisible) {
      setShouldRender(true);
      // Wait for DOM then trigger enter animation
      requestAnimationFrame(() => {
        setAnimationClass('visible');
      });
    } else {
      // Trigger exit animation
      setAnimationClass('');
    }
  }, [isVisible]);

  const handleAnimationEnd = () => {
    if (!isVisible) {
      setShouldRender(false);
    }
  };

  if (!shouldRender) return null;

  return (
    <div
      className={`animated-content ${animationClass}`}
      onTransitionEnd={handleAnimationEnd}
    >
      {children}
    </div>
  );
}
```

```css
.animated-content {
  opacity: 0;
  transform: translateY(10px);
  transition: opacity 300ms, transform 300ms;
}

.animated-content.visible {
  opacity: 1;
  transform: translateY(0);
}
```

## Custom Hook for Enter/Exit

```jsx
function useAnimatedVisibility(isVisible, duration = 300) {
  const [shouldRender, setShouldRender] = useState(isVisible);
  const [isAnimating, setIsAnimating] = useState(false);

  useEffect(() => {
    if (isVisible) {
      setShouldRender(true);
      // Small delay to ensure DOM is ready
      const timer = setTimeout(() => setIsAnimating(true), 10);
      return () => clearTimeout(timer);
    } else {
      setIsAnimating(false);
      const timer = setTimeout(() => setShouldRender(false), duration);
      return () => clearTimeout(timer);
    }
  }, [isVisible, duration]);

  return { shouldRender, isAnimating };
}

// Usage
function Modal({ isOpen, children }) {
  const { shouldRender, isAnimating } = useAnimatedVisibility(isOpen, 300);

  if (!shouldRender) return null;

  return (
    <div className={`modal ${isAnimating ? 'open' : ''}`}>
      {children}
    </div>
  );
}
```

## Using CSS animation-fill-mode

```jsx
function Toast({ message, onClose }) {
  return (
    <div
      className="toast"
      onAnimationEnd={(e) => {
        if (e.animationName === 'fadeOut') {
          onClose();
        }
      }}
    >
      {message}
    </div>
  );
}
```

```css
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(-20px); }
  to { opacity: 1; transform: translateY(0); }
}

@keyframes fadeOut {
  from { opacity: 1; transform: translateY(0); }
  to { opacity: 0; transform: translateY(-20px); }
}

.toast {
  animation: fadeIn 300ms ease-out, fadeOut 300ms ease-out 3s forwards;
}
```

## Animating Lists

```jsx
function AnimatedList({ items }) {
  const [displayItems, setDisplayItems] = useState(items);
  const [removingIds, setRemovingIds] = useState(new Set());

  const removeItem = (id) => {
    setRemovingIds(new Set([...removingIds, id]));
    setTimeout(() => {
      setDisplayItems(items => items.filter(i => i.id !== id));
      setRemovingIds(ids => {
        const newIds = new Set(ids);
        newIds.delete(id);
        return newIds;
      });
    }, 300);
  };

  return (
    <ul>
      {displayItems.map(item => (
        <li
          key={item.id}
          className={removingIds.has(item.id) ? 'removing' : 'visible'}
        >
          {item.name}
          <button onClick={() => removeItem(item.id)}>×</button>
        </li>
      ))}
    </ul>
  );
}
```

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*