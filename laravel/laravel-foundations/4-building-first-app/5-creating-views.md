---
source_course: "laravel-foundations"
source_lesson: "laravel-foundations-creating-views"
---

# Building the Views

Now let's create the Blade templates for our task manager. We'll use Tailwind CSS (included with Breeze) for styling.

## Task Index View

Create `resources/views/tasks/index.blade.php`:

```blade
<x-app-layout>
    <x-slot name="header">
        <div class="flex justify-between items-center">
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                {{ __('My Tasks') }}
            </h2>
            <a href="{{ route('tasks.create') }}" 
               class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded">
                + New Task
            </a>
        </div>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            {{-- Success Message --}}
            @if (session('success'))
                <div class="mb-4 bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded">
                    {{ session('success') }}
                </div>
            @endif

            {{-- Filters --}}
            <div class="mb-6 bg-white overflow-hidden shadow-sm sm:rounded-lg p-4">
                <form method="GET" action="{{ route('tasks.index') }}" class="flex flex-wrap gap-4">
                    <select name="status" class="rounded-md border-gray-300">
                        <option value="">All Status</option>
                        <option value="incomplete" @selected(request('status') === 'incomplete')>Incomplete</option>
                        <option value="completed" @selected(request('status') === 'completed')>Completed</option>
                    </select>
                    
                    <select name="category" class="rounded-md border-gray-300">
                        <option value="">All Categories</option>
                        @foreach ($categories as $category)
                            <option value="{{ $category->id }}" @selected(request('category') == $category->id)>
                                {{ $category->name }}
                            </option>
                        @endforeach
                    </select>
                    
                    <select name="priority" class="rounded-md border-gray-300">
                        <option value="">All Priorities</option>
                        <option value="high" @selected(request('priority') === 'high')>High</option>
                        <option value="medium" @selected(request('priority') === 'medium')>Medium</option>
                        <option value="low" @selected(request('priority') === 'low')>Low</option>
                    </select>
                    
                    <button type="submit" class="bg-gray-800 text-white px-4 py-2 rounded-md">
                        Filter
                    </button>
                    <a href="{{ route('tasks.index') }}" class="text-gray-600 px-4 py-2">
                        Clear
                    </a>
                </form>
            </div>

            {{-- Task List --}}
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                @forelse ($tasks as $task)
                    <div class="p-6 border-b border-gray-200 flex items-center justify-between 
                                @if($task->completed) bg-gray-50 @endif
                                @if($task->isOverdue()) bg-red-50 @endif">
                        <div class="flex items-center gap-4">
                            {{-- Toggle Completion --}}
                            <form method="POST" action="{{ route('tasks.toggle', $task) }}">
                                @csrf
                                @method('PATCH')
                                <button type="submit" 
                                        class="w-6 h-6 rounded-full border-2 flex items-center justify-center
                                               {{ $task->completed ? 'bg-green-500 border-green-500' : 'border-gray-300' }}">
                                    @if ($task->completed)
                                        <svg class="w-4 h-4 text-white" fill="currentColor" viewBox="0 0 20 20">
                                            <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd"/>
                                        </svg>
                                    @endif
                                </button>
                            </form>
                            
                            <div>
                                <a href="{{ route('tasks.show', $task) }}" 
                                   class="text-lg font-medium {{ $task->completed ? 'line-through text-gray-400' : 'text-gray-900' }}">
                                    {{ $task->title }}
                                </a>
                                
                                <div class="flex items-center gap-2 mt-1 text-sm text-gray-500">
                                    @if ($task->category)
                                        <span class="px-2 py-1 rounded text-xs" 
                                              style="background-color: {{ $task->category->color }}20; color: {{ $task->category->color }}">
                                            {{ $task->category->name }}
                                        </span>
                                    @endif
                                    
                                    @if ($task->due_date)
                                        <span class="@if($task->isOverdue()) text-red-600 font-medium @endif">
                                            Due: {{ $task->due_date->format('M d, Y') }}
                                        </span>
                                    @endif
                                    
                                    <span class="px-2 py-1 rounded text-xs
                                        @if($task->priority === 'high') bg-red-100 text-red-800
                                        @elseif($task->priority === 'medium') bg-yellow-100 text-yellow-800
                                        @else bg-green-100 text-green-800
                                        @endif">
                                        {{ ucfirst($task->priority) }}
                                    </span>
                                </div>
                            </div>
                        </div>
                        
                        <div class="flex items-center gap-2">
                            <a href="{{ route('tasks.edit', $task) }}" 
                               class="text-indigo-600 hover:text-indigo-900">Edit</a>
                            
                            <form method="POST" action="{{ route('tasks.destroy', $task) }}" 
                                  onsubmit="return confirm('Are you sure?')">
                                @csrf
                                @method('DELETE')
                                <button type="submit" class="text-red-600 hover:text-red-900">
                                    Delete
                                </button>
                            </form>
                        </div>
                    </div>
                @empty
                    <div class="p-6 text-center text-gray-500">
                        <p>No tasks yet!</p>
                        <a href="{{ route('tasks.create') }}" class="text-indigo-600 hover:underline">
                            Create your first task
                        </a>
                    </div>
                @endforelse
            </div>

            {{-- Pagination --}}
            <div class="mt-4">
                {{ $tasks->links() }}
            </div>
        </div>
    </div>
</x-app-layout>
```

## Task Create View

Create `resources/views/tasks/create.blade.php`:

```blade
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Create Task') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-2xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg p-6">
                <form method="POST" action="{{ route('tasks.store') }}">
                    @csrf
                    
                    {{-- Title --}}
                    <div class="mb-4">
                        <label for="title" class="block text-sm font-medium text-gray-700">Title *</label>
                        <input type="text" 
                               name="title" 
                               id="title" 
                               value="{{ old('title') }}"
                               class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500
                                      @error('title') border-red-500 @enderror"
                               required>
                        @error('title')
                            <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
                        @enderror
                    </div>
                    
                    {{-- Description --}}
                    <div class="mb-4">
                        <label for="description" class="block text-sm font-medium text-gray-700">Description</label>
                        <textarea name="description" 
                                  id="description" 
                                  rows="4"
                                  class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500">{{ old('description') }}</textarea>
                    </div>
                    
                    {{-- Category --}}
                    <div class="mb-4">
                        <label for="category_id" class="block text-sm font-medium text-gray-700">Category</label>
                        <select name="category_id" 
                                id="category_id" 
                                class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500">
                            <option value="">No Category</option>
                            @foreach ($categories as $category)
                                <option value="{{ $category->id }}" @selected(old('category_id') == $category->id)>
                                    {{ $category->name }}
                                </option>
                            @endforeach
                        </select>
                    </div>
                    
                    {{-- Due Date --}}
                    <div class="mb-4">
                        <label for="due_date" class="block text-sm font-medium text-gray-700">Due Date</label>
                        <input type="date" 
                               name="due_date" 
                               id="due_date" 
                               value="{{ old('due_date') }}"
                               min="{{ date('Y-m-d') }}"
                               class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500">
                    </div>
                    
                    {{-- Priority --}}
                    <div class="mb-6">
                        <label for="priority" class="block text-sm font-medium text-gray-700">Priority *</label>
                        <select name="priority" 
                                id="priority" 
                                class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                                required>
                            <option value="low" @selected(old('priority') === 'low')>Low</option>
                            <option value="medium" @selected(old('priority', 'medium') === 'medium')>Medium</option>
                            <option value="high" @selected(old('priority') === 'high')>High</option>
                        </select>
                    </div>
                    
                    {{-- Buttons --}}
                    <div class="flex items-center justify-end gap-4">
                        <a href="{{ route('tasks.index') }}" class="text-gray-600 hover:text-gray-900">
                            Cancel
                        </a>
                        <button type="submit" 
                                class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded">
                            Create Task
                        </button>
                    </div>
                </form>
            </div>
        </div>
    </div>
</x-app-layout>
```

## Dashboard View

Update `resources/views/dashboard.blade.php`:

```blade
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Dashboard') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            {{-- Stats Grid --}}
            <div class="grid grid-cols-1 md:grid-cols-5 gap-4 mb-8">
                <div class="bg-white p-6 rounded-lg shadow">
                    <div class="text-3xl font-bold text-gray-900">{{ $stats['total'] }}</div>
                    <div class="text-gray-500">Total Tasks</div>
                </div>
                <div class="bg-white p-6 rounded-lg shadow">
                    <div class="text-3xl font-bold text-green-600">{{ $stats['completed'] }}</div>
                    <div class="text-gray-500">Completed</div>
                </div>
                <div class="bg-white p-6 rounded-lg shadow">
                    <div class="text-3xl font-bold text-blue-600">{{ $stats['incomplete'] }}</div>
                    <div class="text-gray-500">In Progress</div>
                </div>
                <div class="bg-white p-6 rounded-lg shadow">
                    <div class="text-3xl font-bold text-red-600">{{ $stats['overdue'] }}</div>
                    <div class="text-gray-500">Overdue</div>
                </div>
                <div class="bg-white p-6 rounded-lg shadow">
                    <div class="text-3xl font-bold text-yellow-600">{{ $stats['due_today'] }}</div>
                    <div class="text-gray-500">Due Today</div>
                </div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
                {{-- Recent Tasks --}}
                <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                    <div class="p-6 border-b border-gray-200 font-semibold">
                        Recent Tasks
                    </div>
                    @forelse ($recentTasks as $task)
                        <div class="p-4 border-b border-gray-100 flex items-center justify-between">
                            <div>
                                <a href="{{ route('tasks.show', $task) }}" class="font-medium text-gray-900 hover:text-indigo-600">
                                    {{ $task->title }}
                                </a>
                                @if ($task->category)
                                    <span class="ml-2 text-xs px-2 py-1 rounded" style="background-color: {{ $task->category->color }}20; color: {{ $task->category->color }}">
                                        {{ $task->category->name }}
                                    </span>
                                @endif
                            </div>
                            <span class="text-sm text-gray-500">{{ $task->created_at->diffForHumans() }}</span>
                        </div>
                    @empty
                        <div class="p-4 text-gray-500">No recent tasks</div>
                    @endforelse
                </div>

                {{-- Upcoming Tasks --}}
                <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                    <div class="p-6 border-b border-gray-200 font-semibold">
                        Upcoming Deadlines
                    </div>
                    @forelse ($upcomingTasks as $task)
                        <div class="p-4 border-b border-gray-100 flex items-center justify-between">
                            <div>
                                <a href="{{ route('tasks.show', $task) }}" class="font-medium text-gray-900 hover:text-indigo-600">
                                    {{ $task->title }}
                                </a>
                            </div>
                            <span class="text-sm @if($task->due_date->isToday()) text-yellow-600 font-medium @else text-gray-500 @endif">
                                {{ $task->due_date->format('M d') }}
                                @if($task->due_date->isToday())
                                    (Today)
                                @endif
                            </span>
                        </div>
                    @empty
                        <div class="p-4 text-gray-500">No upcoming deadlines</div>
                    @endforelse
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

You now have a functional task manager! Create the edit view similarly to the create view, and add the show view to display task details.

## Testing Your Application

1. Register a new user at `/register`
2. Create a category
3. Create a few tasks
4. Mark tasks as complete
5. Filter by status, category, or priority
6. View your dashboard statistics

Congratulations! You've built a complete Laravel application! 🎉

## Resources

- [Blade Templates](https://laravel.com/docs/12.x/blade) — Complete Blade templating documentation
- [Tailwind CSS](https://tailwindcss.com/docs) — Tailwind CSS documentation for styling

---

> 📘 *This lesson is part of the [Laravel Foundations](https://stanza.dev/courses/laravel-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*