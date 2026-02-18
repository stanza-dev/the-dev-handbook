---
source_course: "react-testing"
source_lesson: "react-testing-testing-timers"
---

# Testing Timers & Debouncing

Components often use `setTimeout`, `setInterval`, or debouncing. Here's how to test them.

## Jest Fake Timers

```jsx
test('shows message after delay', () => {
  jest.useFakeTimers();
  
  render(<DelayedMessage delay={3000} />);
  
  // Message not visible yet
  expect(screen.queryByText('Hello!')).not.toBeInTheDocument();
  
  // Fast-forward time
  jest.advanceTimersByTime(3000);
  
  // Now visible
  expect(screen.getByText('Hello!')).toBeInTheDocument();
  
  jest.useRealTimers();
});
```

## Testing Debounced Input

```jsx
function SearchBox({ onSearch }) {
  const [query, setQuery] = useState('');
  
  useEffect(() => {
    const timer = setTimeout(() => {
      if (query) onSearch(query);
    }, 300);
    
    return () => clearTimeout(timer);
  }, [query, onSearch]);
  
  return (
    <input
      value={query}
      onChange={e => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}

test('debounces search', async () => {
  jest.useFakeTimers();
  const user = userEvent.setup({ advanceTimers: jest.advanceTimersByTime });
  const onSearch = jest.fn();
  
  render(<SearchBox onSearch={onSearch} />);
  
  await user.type(screen.getByPlaceholderText('Search...'), 'react');
  
  // Not called immediately
  expect(onSearch).not.toHaveBeenCalled();
  
  // Fast-forward past debounce
  jest.advanceTimersByTime(300);
  
  expect(onSearch).toHaveBeenCalledWith('react');
  expect(onSearch).toHaveBeenCalledTimes(1);
  
  jest.useRealTimers();
});
```

## userEvent with Fake Timers

```jsx
// Important: Configure userEvent to use fake timers
const user = userEvent.setup({
  advanceTimers: jest.advanceTimersByTime
});
```

## Testing Intervals

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    return () => clearInterval(interval);
  }, []);
  
  return <span>{seconds}s</span>;
}

test('increments every second', () => {
  jest.useFakeTimers();
  render(<Timer />);
  
  expect(screen.getByText('0s')).toBeInTheDocument();
  
  jest.advanceTimersByTime(1000);
  expect(screen.getByText('1s')).toBeInTheDocument();
  
  jest.advanceTimersByTime(2000);
  expect(screen.getByText('3s')).toBeInTheDocument();
  
  jest.useRealTimers();
});
```

## Run All Timers

```jsx
// Run all pending timers at once
jest.runAllTimers();

// Run only currently pending timers (not new ones created)
jest.runOnlyPendingTimers();
```

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*