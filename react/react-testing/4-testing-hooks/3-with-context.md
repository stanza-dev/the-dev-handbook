---
source_course: "react-testing"
source_lesson: "react-testing-testing-hooks-with-context"
---

# Testing Hooks with Context

Hooks that consume context need a wrapper provider.

## Hook Using Context

```jsx
const AuthContext = createContext(null);

function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  
  const login = async (email, password) => {
    const user = await authApi.login(email, password);
    setUser(user);
  };
  
  const logout = () => setUser(null);
  
  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}
```

## Testing with Wrapper

```jsx
test('useAuth returns user from context', () => {
  const wrapper = ({ children }) => (
    <AuthProvider>{children}</AuthProvider>
  );
  
  const { result } = renderHook(() => useAuth(), { wrapper });
  
  expect(result.current.user).toBe(null);
});

test('login updates user', async () => {
  const wrapper = ({ children }) => (
    <AuthProvider>{children}</AuthProvider>
  );
  
  const { result } = renderHook(() => useAuth(), { wrapper });
  
  await act(async () => {
    await result.current.login('test@example.com', 'password');
  });
  
  expect(result.current.user).toEqual({
    email: 'test@example.com'
  });
});
```

## Custom Wrapper with Props

```jsx
function createWrapper(initialUser = null) {
  return function Wrapper({ children }) {
    return (
      <AuthProvider initialUser={initialUser}>
        {children}
      </AuthProvider>
    );
  };
}

test('with authenticated user', () => {
  const mockUser = { id: '1', name: 'John' };
  const wrapper = createWrapper(mockUser);
  
  const { result } = renderHook(() => useAuth(), { wrapper });
  
  expect(result.current.user).toEqual(mockUser);
});
```

## Testing Error Cases

```jsx
test('throws error when used outside provider', () => {
  // Suppress console.error for this test
  const consoleSpy = jest.spyOn(console, 'error').mockImplementation();
  
  expect(() => {
    renderHook(() => useAuth());
  }).toThrow('useAuth must be used within AuthProvider');
  
  consoleSpy.mockRestore();
});
```

## Reusable Test Utility

```jsx
// test-utils.js
export function renderHookWithProviders(hook, options = {}) {
  const wrapper = ({ children }) => (
    <AuthProvider>
      <ThemeProvider>
        <QueryClientProvider client={queryClient}>
          {children}
        </QueryClientProvider>
      </ThemeProvider>
    </AuthProvider>
  );
  
  return renderHook(hook, { wrapper, ...options });
}
```

---

> 📘 *This lesson is part of the [React Testing Strategies](https://stanza.dev/courses/react-testing) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*