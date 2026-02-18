---
source_course: "vue-pinia"
source_lesson: "vue-pinia-async-actions"
---

# Async Actions and API Integration

Async actions are crucial for API calls, data fetching, and any asynchronous operations. Pinia makes them straightforward.

## Basic Async Action

```typescript
import { defineStore } from 'pinia'

type User = {
  id: number
  name: string
  email: string
}

export const useUserStore = defineStore('user', {
  state: () => ({
    user: null as User | null,
    loading: false,
    error: null as string | null
  }),
  
  actions: {
    async fetchUser(id: number) {
      this.loading = true
      this.error = null
      
      try {
        const response = await fetch(`/api/users/${id}`)
        
        if (!response.ok) {
          throw new Error('User not found')
        }
        
        this.user = await response.json()
      } catch (e) {
        this.error = (e as Error).message
        this.user = null
      } finally {
        this.loading = false
      }
    }
  }
})
```

## CRUD Operations Store

```typescript
import { defineStore } from 'pinia'

type Todo = {
  id: number
  title: string
  completed: boolean
}

export const useTodoStore = defineStore('todo', {
  state: () => ({
    todos: [] as Todo[],
    loading: false,
    error: null as string | null
  }),
  
  getters: {
    completedTodos: (state) => state.todos.filter(t => t.completed),
    pendingTodos: (state) => state.todos.filter(t => !t.completed),
    todoById: (state) => (id: number) => state.todos.find(t => t.id === id)
  },
  
  actions: {
    // READ
    async fetchTodos() {
      this.loading = true
      try {
        const response = await fetch('/api/todos')
        this.todos = await response.json()
      } catch (e) {
        this.error = 'Failed to fetch todos'
      } finally {
        this.loading = false
      }
    },
    
    // CREATE
    async addTodo(title: string) {
      const optimisticTodo: Todo = {
        id: Date.now(),  // Temporary ID
        title,
        completed: false
      }
      
      // Optimistic update
      this.todos.push(optimisticTodo)
      
      try {
        const response = await fetch('/api/todos', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ title })
        })
        
        const serverTodo = await response.json()
        
        // Replace optimistic todo with server response
        const index = this.todos.findIndex(t => t.id === optimisticTodo.id)
        if (index !== -1) {
          this.todos[index] = serverTodo
        }
        
        return serverTodo
      } catch (e) {
        // Rollback on error
        this.todos = this.todos.filter(t => t.id !== optimisticTodo.id)
        throw e
      }
    },
    
    // UPDATE
    async toggleTodo(id: number) {
      const todo = this.todos.find(t => t.id === id)
      if (!todo) return
      
      const previousState = todo.completed
      
      // Optimistic update
      todo.completed = !todo.completed
      
      try {
        await fetch(`/api/todos/${id}`, {
          method: 'PATCH',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ completed: todo.completed })
        })
      } catch (e) {
        // Rollback
        todo.completed = previousState
        throw e
      }
    },
    
    // DELETE
    async deleteTodo(id: number) {
      const index = this.todos.findIndex(t => t.id === id)
      if (index === -1) return
      
      const deleted = this.todos[index]
      
      // Optimistic delete
      this.todos.splice(index, 1)
      
      try {
        await fetch(`/api/todos/${id}`, { method: 'DELETE' })
      } catch (e) {
        // Rollback
        this.todos.splice(index, 0, deleted)
        throw e
      }
    }
  }
})
```

## Parallel and Sequential Requests

```typescript
actions: {
  // Parallel requests
  async fetchDashboardData() {
    this.loading = true
    
    try {
      const [users, orders, stats] = await Promise.all([
        fetch('/api/users').then(r => r.json()),
        fetch('/api/orders').then(r => r.json()),
        fetch('/api/stats').then(r => r.json())
      ])
      
      this.users = users
      this.orders = orders
      this.stats = stats
    } finally {
      this.loading = false
    }
  },
  
  // Sequential requests
  async createOrderWithItems(orderData: OrderData, items: Item[]) {
    this.loading = true
    
    try {
      // First create the order
      const order = await fetch('/api/orders', {
        method: 'POST',
        body: JSON.stringify(orderData)
      }).then(r => r.json())
      
      // Then add items to the order
      for (const item of items) {
        await fetch(`/api/orders/${order.id}/items`, {
          method: 'POST',
          body: JSON.stringify(item)
        })
      }
      
      return order
    } finally {
      this.loading = false
    }
  }
}
```

## Debouncing API Calls

```typescript
import { defineStore } from 'pinia'
import debounce from 'lodash/debounce'

export const useSearchStore = defineStore('search', {
  state: () => ({
    query: '',
    results: [] as SearchResult[],
    loading: false
  }),
  
  actions: {
    // Debounced search
    search: debounce(async function (this: any, query: string) {
      if (!query.trim()) {
        this.results = []
        return
      }
      
      this.loading = true
      this.query = query
      
      try {
        const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`)
        this.results = await response.json()
      } finally {
        this.loading = false
      }
    }, 300),
    
    clearSearch() {
      this.query = ''
      this.results = []
    }
  }
})
```

## Resources

- [Actions](https://pinia.vuejs.org/core-concepts/actions.html) — More on Pinia actions

---

> 📘 *This lesson is part of the [State Management with Pinia](https://stanza.dev/courses/vue-pinia) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*