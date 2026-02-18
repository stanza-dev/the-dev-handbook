---
source_course: "react-animation"
source_lesson: "react-animation-css-keyframes"
---

# CSS Keyframe Animations

Keyframes allow multi-step animations.

## Basic Keyframes

```css
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes slideUp {
  0% {
    opacity: 0;
    transform: translateY(20px);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}

.fade-in {
  animation: fadeIn 300ms ease-out forwards;
}

.slide-up {
  animation: slideUp 400ms ease-out forwards;
}
```

## Multi-Step Animations

```css
@keyframes bounce {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-20px);
  }
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}

.bounce {
  animation: bounce 1s ease-in-out infinite;
}

.pulse {
  animation: pulse 2s ease-in-out infinite;
}
```

## Animation Properties

```css
.element {
  animation-name: slideUp;
  animation-duration: 400ms;
  animation-timing-function: ease-out;
  animation-delay: 100ms;
  animation-iteration-count: 1;      /* or infinite */
  animation-direction: normal;        /* or reverse, alternate */
  animation-fill-mode: forwards;      /* Keep final state */
  animation-play-state: running;      /* or paused */
}

/* Shorthand */
.element {
  animation: slideUp 400ms ease-out 100ms 1 normal forwards;
}
```

## Triggering with React State

```jsx
function Notification({ message, isVisible }) {
  if (!isVisible) return null;
  
  return (
    <div className="notification slide-in">
      {message}
    </div>
  );
}
```

```css
@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

.slide-in {
  animation: slideIn 300ms ease-out forwards;
}
```

## Loading Spinner

```jsx
function Spinner() {
  return <div className="spinner" />;
}
```

```css
@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.spinner {
  width: 24px;
  height: 24px;
  border: 3px solid #e5e7eb;
  border-top-color: #3b82f6;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}
```

## Staggered Animations

```jsx
function List({ items }) {
  return (
    <ul>
      {items.map((item, index) => (
        <li
          key={item.id}
          className="list-item"
          style={{ animationDelay: `${index * 100}ms` }}
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

```css
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.list-item {
  opacity: 0;
  animation: fadeInUp 300ms ease-out forwards;
}
```

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*