---
source_course: "react-testing"
source_lesson: "react-testing-common-mistakes"
---

# Common Testing Mistakes

Avoid these pitfalls to write better tests.

## 1. Testing Implementation Details

```jsx
// ❌ Bad: Testing internal state
test('sets loading to true', () => {
  const { result } = renderHook(() => useState(false));
  // Testing useState itself, not behavior
});

// ✅ Good: Testing user-visible outcome
test('shows spinner while loading', async () => {
  render(<DataLoader />);
  await user.click(screen.getByText('Load'));
  expect(screen.getByRole('status')).toBeInTheDocument();
});
```

## 2. Using Wrong Query

```jsx
// ❌ Bad: Using getBy for absent element
test('error not shown initially', () => {
  render(<Form />);
  expect(screen.getByText('Error')).not.toBeInTheDocument();
  // Throws! getBy doesn't return null
});

// ✅ Good: Use queryBy for absent elements
test('error not shown initially', () => {
  render(<Form />);
  expect(screen.queryByText('Error')).not.toBeInTheDocument();
});
```

## 3. Not Waiting for Async

```jsx
// ❌ Bad: Not waiting for async update
test('shows data', () => {
  render(<AsyncComponent />);
  expect(screen.getByText('Data')).toBeInTheDocument();
  // Fails! Data hasn't loaded yet
});

// ✅ Good: Wait for element
test('shows data', async () => {
  render(<AsyncComponent />);
  expect(await screen.findByText('Data')).toBeInTheDocument();
});
```

## 4. Incorrect waitFor Usage

```jsx
// ❌ Bad: Side effects in waitFor
await waitFor(() => {
  fireEvent.click(button); // Don't do this!
  expect(result).toBe(true);
});

// ✅ Good: Only assertions in waitFor
await user.click(button);
await waitFor(() => {
  expect(result).toBe(true);
});
```

## 5. Snapshot Overuse

```jsx
// ❌ Bad: Large, fragile snapshot
test('renders correctly', () => {
  const { container } = render(<ComplexPage />);
  expect(container).toMatchSnapshot();
  // Breaks on any change, even valid ones
});

// ✅ Good: Targeted assertions
test('renders header with user name', () => {
  render(<ComplexPage user={{ name: 'John' }} />);
  expect(screen.getByRole('heading')).toHaveTextContent('Welcome, John');
});
```

## 6. Not Cleaning Up

```jsx
// ❌ Bad: Global state leaking between tests
let mockData = [];

test('adds item', () => {
  mockData.push('item');
  // Next test starts with ['item']!
});

// ✅ Good: Reset in beforeEach/afterEach
beforeEach(() => {
  mockData = [];
});
```

## 7. Hardcoded Timeouts

```jsx
// ❌ Bad: Arbitrary wait time
test('shows success', async () => {
  render(<Form />);
  await user.click(submitButton);
  await new Promise(r => setTimeout(r, 2000)); // Slow and flaky!
  expect(screen.getByText('Success')).toBeInTheDocument();
});

// ✅ Good: Wait for actual condition
test('shows success', async () => {
  render(<Form />);
  await user.click(submitButton);
  await screen.findByText('Success');
});
```

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*