---
source_course: "react-project-architecture"
source_lesson: "react-project-architecture-api-client"
---

# API Client Design

Create a structured API layer.

## Basic API Client

```tsx
// api/client.ts
const BASE_URL = process.env.NEXT_PUBLIC_API_URL;

class APIClient {
  private baseUrl: string;
  private token: string | null = null;
  
  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }
  
  setToken(token: string | null) {
    this.token = token;
  }
  
  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const headers: HeadersInit = {
      'Content-Type': 'application/json',
      ...(this.token && { Authorization: `Bearer ${this.token}` }),
      ...options.headers,
    };
    
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      ...options,
      headers,
    });
    
    if (!response.ok) {
      const error = await response.json().catch(() => ({}));
      throw new APIError(response.status, error.message || 'Request failed');
    }
    
    return response.json();
  }
  
  get<T>(endpoint: string) {
    return this.request<T>(endpoint);
  }
  
  post<T>(endpoint: string, data: unknown) {
    return this.request<T>(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
    });
  }
  
  put<T>(endpoint: string, data: unknown) {
    return this.request<T>(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data),
    });
  }
  
  delete<T>(endpoint: string) {
    return this.request<T>(endpoint, { method: 'DELETE' });
  }
}

export const apiClient = new APIClient(BASE_URL);
```

## Resource APIs

```tsx
// api/resources/users.ts
import { apiClient } from '../client';
import type { User, CreateUserInput, UpdateUserInput } from '@/types';

export const usersApi = {
  getAll: () => 
    apiClient.get<User[]>('/users'),
  
  getById: (id: string) => 
    apiClient.get<User>(`/users/${id}`),
  
  create: (data: CreateUserInput) => 
    apiClient.post<User>('/users', data),
  
  update: (id: string, data: UpdateUserInput) => 
    apiClient.put<User>(`/users/${id}`, data),
  
  delete: (id: string) => 
    apiClient.delete<void>(`/users/${id}`),
};
```

## API Index

```tsx
// api/index.ts
export { apiClient } from './client';
export { usersApi } from './resources/users';
export { productsApi } from './resources/products';
export { ordersApi } from './resources/orders';
```

## With TanStack Query

```tsx
// hooks/useUsers.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { usersApi } from '@/api';

export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: usersApi.getAll,
  });
}

export function useUser(id: string) {
  return useQuery({
    queryKey: ['users', id],
    queryFn: () => usersApi.getById(id),
    enabled: !!id,
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: usersApi.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}
```

## Type-Safe API with Zod

```tsx
import { z } from 'zod';

const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
  email: z.string().email(),
});

type User = z.infer<typeof UserSchema>;

const usersApi = {
  getById: async (id: string): Promise<User> => {
    const data = await apiClient.get(`/users/${id}`);
    return UserSchema.parse(data); // Runtime validation
  },
};
```

---

> 📘 *This lesson is part of the [React Project Architecture](https://stanza.dev/courses/react-project-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*