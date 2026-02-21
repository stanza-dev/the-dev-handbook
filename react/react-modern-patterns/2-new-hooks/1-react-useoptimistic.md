---
source_course: "react-modern-patterns"
source_lesson: "react-useoptimistic"
---

# useOptimistic for Instant UI Updates

## Introduction

Users expect instant feedback when they interact with your app. Clicking a "Like" button and waiting two seconds for the heart to fill feels broken, even if the server needs that time to process the request. The `useOptimistic` hook in React 19 solves this by letting you show an immediate UI update while the actual async operation happens in the background. If the operation fails, React automatically reverts the optimistic state.

## Key Concepts

- **Optimistic Update**: A UI change shown immediately before server confirmation, assuming the operation will succeed.
- **`useOptimistic(state, updateFn)`**: A hook that takes the current actual state and an update function, returning the optimistic state and a function to trigger optimistic updates.
- **Automatic Revert**: If the parent Action or transition fails, the optimistic value reverts to the real state automatically.
- **Pending Indicator**: You can add a flag like `sending: true` to optimistically added items to show the user that the operation is still in progress.

## Real World Context

Consider a social media feed where users can like posts. When a user taps the heart icon, you want the heart to fill immediately and the like count to increment. Behind the scenes, an API call saves the like to the server. If the server responds with an error (maybe the user's session expired), the heart unfills and the count decrements automatically. This pattern is used by every major social platform.

## Deep Dive

The hook signature is straightforward:

```tsx
const [optimisticState, addOptimistic] = useOptimistic(actualState, updateFn);
```

The `updateFn` receives the current optimistic state and the value you pass to `addOptimistic`, returning the new optimistic state. Here is a complete example with a message list:

```tsx
import { useOptimistic, useState, useRef } from "react";

type Message = {
  id: string;
  text: string;
  sending?: boolean;
};

function Chat({ initialMessages }: { initialMessages: Message[] }) {
  const [messages, setMessages] = useState(initialMessages);
  const formRef = useRef<HTMLFormElement>(null);

  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (currentMessages: Message[], newText: string) => [
      ...currentMessages,
      { id: `temp-${Date.now()}`, text: newText, sending: true },
    ]
  );

  async function sendMessage(formData: FormData) {
    const text = formData.get("message") as string;
    addOptimisticMessage(text);
    formRef.current?.reset();

    const saved = await saveMessage(text);
    setMessages((prev) => [
      ...prev,
      { id: saved.id, text: saved.text },
    ]);
  }

  return (
    <div>
      {optimisticMessages.map((msg) => (
        <div key={msg.id} className={msg.sending ? "opacity-50" : ""}>
          {msg.text}
          {msg.sending && <span> (Sending...)</span>}
        </div>
      ))}
      <form ref={formRef} action={sendMessage}>
        <input name="message" placeholder="Type a message" />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

The optimistic state exists only during the transition. Once the Action completes and `setMessages` updates the real state, the optimistic layer is discarded and replaced by the real data.

## Common Pitfalls

1. **Using `useOptimistic` outside a transition** — The hook only works when the optimistic update is triggered inside an Action or transition. Calling `addOptimistic` in a plain event handler will not revert on failure.
2. **Mutating the state array** — The update function must return a new array or object, not mutate the existing one. Using `push()` instead of spread will cause stale UI bugs.

## Best Practices

1. **Add a visual indicator for optimistic items** — Use a `sending` or `pending` flag on optimistic entries so users can distinguish between confirmed and in-flight data. A subtle opacity change or spinner works well.
2. **Keep optimistic updates simple** — Only update what the user expects to see change. Do not try to optimistically update computed fields like totals or rankings that depend on server-side logic.

## Summary

- `useOptimistic` shows immediate UI changes while async operations run in the background.
- The hook takes the real state and an update function, returning optimistic state and a trigger function.
- React automatically reverts optimistic updates if the Action fails, so you do not need manual rollback logic.

## Code Examples

**Todo list with optimistic additions using useOptimistic**

```tsx
import { useOptimistic, useState } from "react";

type Todo = { id: string; text: string; sending?: boolean };

function TodoList({ initial }: { initial: Todo[] }) {
  const [todos, setTodos] = useState(initial);
  const [optimisticTodos, addOptimistic] = useOptimistic(
    todos,
    (current: Todo[], newText: string) => [
      ...current,
      { id: "temp-" + Date.now(), text: newText, sending: true },
    ]
  );

  async function addTodo(formData: FormData) {
    const text = formData.get("text") as string;
    addOptimistic(text);
    const saved = await saveTodo(text);
    setTodos((prev) => [...prev, { id: saved.id, text: saved.text }]);
  }

  return (
    <div>
      {optimisticTodos.map((t) => (
        <p key={t.id} style={{ opacity: t.sending ? 0.5 : 1 }}>{t.text}</p>
      ))}
      <form action={addTodo}>
        <input name="text" />
        <button type="submit">Add</button>
      </form>
    </div>
  );
}
```


## Resources

- [useOptimistic Reference](https://react.dev/reference/react/useOptimistic) — Official useOptimistic hook documentation
- [React 19 Blog Post](https://react.dev/blog/2024/12/05/react-19) — Official React 19 announcement covering new hooks

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*