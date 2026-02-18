---
source_course: "react-typescript"
source_lesson: "react-typescript-typing-api-responses"
---

# Typing API Responses

Type-safe API calls and responses.

## Basic Fetch Typing

```tsx
type User = {
  id: string;
  name: string;
  email: string;
};

async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    throw new Error('Failed to fetch');
  }
  return response.json() as Promise<User>;
}
```

## Generic Fetch Wrapper

```tsx
async function api<T>(url: string, options?: RequestInit): Promise<T> {
  const response = await fetch(url, {
    headers: {
      'Content-Type': 'application/json',
      ...options?.headers,
    },
    ...options,
  });

  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }

  return response.json() as Promise<T>;
}

// Usage
const user = await api<User>('/api/users/1');
const users = await api<User[]>('/api/users');
```

## With Zod Validation

```tsx
import { z } from 'zod';

// Define schema
const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email(),
  createdAt: z.string().transform(s => new Date(s)),
});

// Infer type from schema
type User = z.infer<typeof UserSchema>;

async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();
  
  // Runtime validation + type narrowing
  return UserSchema.parse(data);
}
```

## TanStack Query with Types

```tsx
import { useQuery, useMutation } from '@tanstack/react-query';

type User = {
  id: string;
  name: string;
};

type CreateUserInput = {
  name: string;
  email: string;
};

// Typed query
function useUser(id: string) {
  return useQuery<User, Error>({
    queryKey: ['user', id],
    queryFn: () => api<User>(`/api/users/${id}`),
  });
}

// Typed mutation
function useCreateUser() {
  return useMutation<User, Error, CreateUserInput>({
    mutationFn: (data) => api<User>('/api/users', {
      method: 'POST',
      body: JSON.stringify(data),
    }),
  });
}
```

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*