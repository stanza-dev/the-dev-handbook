---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-useimperativehandle"
---

# useImperativeHandle: Exposing Custom Methods

`useImperativeHandle` lets you customize the handle exposed as a ref. This is useful when you want to expose a limited, imperative API to parent components.

## Basic Usage

```tsx
import { useRef, useImperativeHandle, forwardRef } from 'react';

type VideoPlayerHandle = {
  play: () => void;
  pause: () => void;
  seekTo: (time: number) => void;
};

const VideoPlayer = forwardRef<VideoPlayerHandle, VideoPlayerProps>(
  function VideoPlayer({ src }, ref) {
    const videoRef = useRef<HTMLVideoElement>(null);

    useImperativeHandle(ref, () => ({
      play() {
        videoRef.current?.play();
      },
      pause() {
        videoRef.current?.pause();
      },
      seekTo(time: number) {
        if (videoRef.current) {
          videoRef.current.currentTime = time;
        }
      }
    }), []);

    return <video ref={videoRef} src={src} />;
  }
);

// Parent component
function App() {
  const playerRef = useRef<VideoPlayerHandle>(null);

  return (
    <>
      <VideoPlayer ref={playerRef} src="/video.mp4" />
      <button onClick={() => playerRef.current?.play()}>Play</button>
      <button onClick={() => playerRef.current?.pause()}>Pause</button>
      <button onClick={() => playerRef.current?.seekTo(30)}>Skip to 30s</button>
    </>
  );
}
```

## React 19: Ref as a Prop

In React 19, `ref` is a regular prop, so `forwardRef` is no longer required:

```tsx
function VideoPlayer({ src, ref }: VideoPlayerProps & { ref?: Ref<VideoPlayerHandle> }) {
  const videoRef = useRef<HTMLVideoElement>(null);

  useImperativeHandle(ref, () => ({
    play: () => videoRef.current?.play(),
    pause: () => videoRef.current?.pause(),
    seekTo: (time) => {
      if (videoRef.current) videoRef.current.currentTime = time;
    }
  }), []);

  return <video ref={videoRef} src={src} />;
}
```

## Real-World Example: Form with Focus Management

```tsx
type FormHandle = {
  focusFirstError: () => void;
  reset: () => void;
  submit: () => void;
};

function Form({ ref, onSubmit, children }: FormProps) {
  const formRef = useRef<HTMLFormElement>(null);
  const [errors, setErrors] = useState<Record<string, string>>({});

  useImperativeHandle(ref, () => ({
    focusFirstError() {
      const firstErrorField = Object.keys(errors)[0];
      if (firstErrorField) {
        const element = formRef.current?.elements.namedItem(firstErrorField);
        (element as HTMLElement)?.focus();
      }
    },
    reset() {
      formRef.current?.reset();
      setErrors({});
    },
    submit() {
      formRef.current?.requestSubmit();
    }
  }), [errors]);

  return (
    <form ref={formRef} onSubmit={onSubmit}>
      {children}
    </form>
  );
}
```

## When to Use

✅ Exposing imperative methods (focus, scroll, play/pause)
✅ Hiding internal implementation details
✅ Creating reusable component libraries

❌ Don't use for data that could be props
❌ Avoid overusing - prefer declarative patterns

## Resources

- [useImperativeHandle API Reference](https://react.dev/reference/react/useImperativeHandle) — Official React documentation for useImperativeHandle hook

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*