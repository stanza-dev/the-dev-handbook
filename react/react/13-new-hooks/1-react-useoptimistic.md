---
source_course: "react"
source_lesson: "react-useoptimistic"
---

# useOptimistic Hook

Show changes instantly while waiting for server confirmation. If the operation fails, the UI reverts automatically.

## Signature

```jsx
const [optimisticState, addOptimistic] = useOptimistic(
  state,
  updateFn
);
```

## Use Cases

- Like/unlike buttons
- Adding items to lists
- Toggle switches
- Any action where immediate feedback improves UX

## How It Works

1. User triggers action → UI updates immediately
2. Server request happens in background
3. On success → optimistic state becomes real state
4. On failure → UI reverts to previous state

## Code Examples

**Chat with optimistic message sending**

```tsx
import { useOptimistic, useState, useRef } from 'react';

type Message = {
  id: string;
  text: string;
  sending?: boolean;
};

function Chat({ initialMessages }: { initialMessages: Message[] }) {
  const [messages, setMessages] = useState(initialMessages);
  const formRef = useRef<HTMLFormElement>(null);
  
  // Optimistic state shows messages immediately
  const [optimisticMessages, addOptimisticMessage] = useOptimistic(
    messages,
    (currentMessages, newText: string) => [
      ...currentMessages,
      { 
        id: `temp-${Date.now()}`, 
        text: newText, 
        sending: true  // Visual indicator
      }
    ]
  );

  async function sendMessage(formData: FormData) {
    const text = formData.get('message') as string;
    
    // Show immediately in UI
    addOptimisticMessage(text);
    formRef.current?.reset();
    
    // Send to server
    const savedMessage = await saveMessage(text);
    
    // Update with real server response
    setMessages(prev => [
      ...prev, 
      { id: savedMessage.id, text: savedMessage.text }
    ]);
  }

  return (
    <div className="chat">
      {optimisticMessages.map(msg => (
        <div key={msg.id} className={msg.sending ? 'sending' : ''}>
          {msg.text}
          {msg.sending && <span className="indicator">Sending...</span>}
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


## Resources

- [React 19 Blog Post](https://react.dev/blog/2024/12/05/react-19) — Official React 19 announcement

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*