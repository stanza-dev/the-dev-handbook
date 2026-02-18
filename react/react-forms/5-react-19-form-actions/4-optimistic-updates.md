---
source_course: "react-forms"
source_lesson: "react-forms-optimistic-updates"
---

# Optimistic Updates with useOptimistic

`useOptimistic` lets you show a UI update immediately while the actual action is still pending. If the action fails, the UI reverts to the previous state.

## Why Optimistic Updates?

Instead of:
1. User clicks → Loading spinner → Success message

You get:
1. User clicks → Instant UI update → (Background: server confirms)

This makes apps feel much faster!

## Basic Syntax

```jsx
const [optimisticState, addOptimistic] = useOptimistic(
  currentState,
  updateFunction
);
```

## Example: Like Button

```jsx
import { useOptimistic } from 'react';

function LikeButton({ likes, postId }) {
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    likes,
    (currentLikes, increment) => currentLikes + increment
  );

  async function handleLike() {
    addOptimisticLike(1); // Immediately show +1
    await likePost(postId); // Actually like the post
  }

  return (
    <form action={handleLike}>
      <button type="submit">❤️ {optimisticLikes}</button>
    </form>
  );
}
```

## Example: Todo List

```jsx
function TodoList({ todos }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (currentTodos, newTodo) => [
      ...currentTodos,
      { ...newTodo, sending: true } // Mark as pending
    ]
  );

  async function addTodo(formData) {
    const text = formData.get('text');
    const newTodo = { id: Date.now(), text, completed: false };
    
    addOptimisticTodo(newTodo); // Show immediately
    await saveTodo(newTodo); // Save to server
  }

  return (
    <>
      <form action={addTodo}>
        <input name="text" placeholder="New todo" />
        <button type="submit">Add</button>
      </form>
      
      <ul>
        {optimisticTodos.map(todo => (
          <li 
            key={todo.id}
            style={{ opacity: todo.sending ? 0.5 : 1 }}
          >
            {todo.text}
            {todo.sending && ' (saving...)'}
          </li>
        ))}
      </ul>
    </>
  );
}
```

## Handling Errors

If the action fails, the optimistic state automatically reverts:

```jsx
async function addTodo(formData) {
  const text = formData.get('text');
  addOptimisticTodo({ id: Date.now(), text });
  
  try {
    await saveTodo(text);
  } catch (error) {
    // The optimistic state will revert automatically
    // when the action completes (even with error)
    toast.error('Failed to add todo');
  }
}
```

📚 **Learn more**: [useOptimistic Reference](https://react.dev/reference/react/useOptimistic)

---

> 📘 *This lesson is part of the [React Form Mastery](https://stanza.dev/courses/react-forms) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*