---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-r19-useoptimistic"
---

# useOptimistic: Optimistic UI Updates

`useOptimistic` lets you show a different state while an async action is in progress. This creates a snappier user experience by assuming success.

## Basic Usage

```tsx
import { useOptimistic } from 'react';

type Message = {
  id: string;
  text: string;
  sending?: boolean;
};

function Chat({ messages }: { messages: Message[] }) {
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (state, newMessage: string) => [
      ...state,
      { id: crypto.randomUUID(), text: newMessage, sending: true }
    ]
  );

  async function sendMessage(formData: FormData) {
    const text = formData.get('message') as string;
    addOptimisticMessage(text); // Show immediately
    await deliverMessage(text); // Actually send
  }

  return (
    <>
      <ul>
        {optimisticMessages.map((msg) => (
          <li key={msg.id} style={{ opacity: msg.sending ? 0.5 : 1 }}>
            {msg.text}
            {msg.sending && ' (sending...)'}
          </li>
        ))}
      </ul>
      <form action={sendMessage}>
        <input name="message" />
        <button>Send</button>
      </form>
    </>
  );
}
```

## How It Works

1. **Initial state**: `optimisticState` equals the actual state
2. **Action starts**: Call `addOptimistic(value)` to show optimistic state
3. **During action**: `optimisticState` shows the optimistic value
4. **Action completes**: `optimisticState` reverts to actual state (which should now include the change)

## Like Button Example

```tsx
function LikeButton({ post }: { post: Post }) {
  const [optimisticLiked, setOptimisticLiked] = useOptimistic(
    post.isLiked,
    (_, newValue: boolean) => newValue
  );

  async function toggleLike() {
    setOptimisticLiked(!optimisticLiked);
    await updateLikeStatus(post.id, !post.isLiked);
  }

  return (
    <button onClick={toggleLike}>
      {optimisticLiked ? '❤️' : '🤍'}
    </button>
  );
}
```

## Error Handling

If the action fails, the optimistic state automatically reverts:

```tsx
async function sendMessage(formData: FormData) {
  const text = formData.get('message') as string;
  addOptimisticMessage(text);
  
  try {
    await deliverMessage(text);
  } catch (error) {
    // Optimistic state automatically reverts!
    // Show error to user
    toast.error('Failed to send message');
  }
}
```

## Best Practices

- Use for actions likely to succeed
- Show visual indicators for pending state
- Handle errors gracefully
- Keep optimistic updates simple

## Resources

- [useOptimistic API Reference](https://react.dev/reference/react/useOptimistic) — Official React documentation for useOptimistic hook

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*