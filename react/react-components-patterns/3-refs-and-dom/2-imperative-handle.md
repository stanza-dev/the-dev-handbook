---
source_course: "react-components-patterns"
source_lesson: "react-components-patterns-imperative-handle"
---

# useImperativeHandle: Custom Ref APIs

`useImperativeHandle` lets you customize what value is exposed when a parent uses a ref on your component.

## Why Customize Refs?

Instead of exposing the raw DOM node, expose a limited, intentional API:

```tsx
// Without useImperativeHandle - exposes entire DOM node
function Input({ ref }) {
  return <input ref={ref} />;
}
// Parent can do ANYTHING: ref.current.style.color = 'red'

// With useImperativeHandle - controlled API
function Input({ ref }) {
  const inputRef = useRef<HTMLInputElement>(null);
  
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current?.focus(),
    clear: () => {
      if (inputRef.current) inputRef.current.value = '';
    }
  }));
  
  return <input ref={inputRef} />;
}
// Parent can only: ref.current.focus() or ref.current.clear()
```

## Video Player Example

```tsx
type VideoPlayerHandle = {
  play: () => void;
  pause: () => void;
  seek: (time: number) => void;
  getCurrentTime: () => number;
};

type VideoPlayerProps = {
  src: string;
  ref?: React.Ref<VideoPlayerHandle>;
};

function VideoPlayer({ src, ref }: VideoPlayerProps) {
  const videoRef = useRef<HTMLVideoElement>(null);
  
  useImperativeHandle(ref, () => ({
    play() {
      videoRef.current?.play();
    },
    pause() {
      videoRef.current?.pause();
    },
    seek(time: number) {
      if (videoRef.current) {
        videoRef.current.currentTime = time;
      }
    },
    getCurrentTime() {
      return videoRef.current?.currentTime ?? 0;
    }
  }), []);
  
  return <video ref={videoRef} src={src} />;
}

// Usage
function App() {
  const playerRef = useRef<VideoPlayerHandle>(null);
  
  return (
    <div>
      <VideoPlayer ref={playerRef} src="/video.mp4" />
      <button onClick={() => playerRef.current?.play()}>Play</button>
      <button onClick={() => playerRef.current?.pause()}>Pause</button>
      <button onClick={() => playerRef.current?.seek(30)}>Skip to 30s</button>
    </div>
  );
}
```

## Form with Validation

```tsx
type FormHandle = {
  submit: () => void;
  reset: () => void;
  focusFirstError: () => void;
  isValid: () => boolean;
};

function Form({ ref, onSubmit, children }: FormProps) {
  const formRef = useRef<HTMLFormElement>(null);
  const [errors, setErrors] = useState<Record<string, string>>({});
  
  useImperativeHandle(ref, () => ({
    submit() {
      formRef.current?.requestSubmit();
    },
    reset() {
      formRef.current?.reset();
      setErrors({});
    },
    focusFirstError() {
      const firstError = Object.keys(errors)[0];
      if (firstError) {
        const element = formRef.current?.elements.namedItem(firstError);
        (element as HTMLElement)?.focus();
      }
    },
    isValid() {
      return Object.keys(errors).length === 0;
    }
  }), [errors]);
  
  return (
    <form ref={formRef} onSubmit={onSubmit}>
      {children}
    </form>
  );
}
```

## Best Practices

1. **Expose minimal API** - Only what's truly needed
2. **Use descriptive method names** - `play()` not `p()`
3. **Include dependencies** - Pass deps array to useImperativeHandle
4. **Document the API** - TypeScript types help!

```tsx
// ❌ Too much exposure
useImperativeHandle(ref, () => ({
  getNode: () => inputRef.current, // Exposes everything!
}));

// ✅ Intentional API
useImperativeHandle(ref, () => ({
  focus: () => inputRef.current?.focus(),
  blur: () => inputRef.current?.blur(),
  getValue: () => inputRef.current?.value ?? '',
}));
```

## Resources

- [useImperativeHandle](https://react.dev/reference/react/useImperativeHandle) — Official React documentation for useImperativeHandle

---

> 📘 *This lesson is part of the [React Components & Patterns](https://stanza.dev/courses/react-components-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*