---
source_course: "laravel-foundations"
source_lesson: "laravel-foundations-routes-controllers"
---

# Creating Routes and Controllers

Now let's connect our models to the web by creating routes and controllers.

## Planning Our Routes

We need these routes for our task manager:

| Method | URI | Action | Description |
|--------|-----|--------|-------------|
| GET | /dashboard | index | Show dashboard with stats |
| GET | /tasks | index | List all tasks |
| GET | /tasks/create | create | Show create form |
| POST | /tasks | store | Save new task |
| GET | /tasks/{task} | show | Show single task |
| GET | /tasks/{task}/edit | edit | Show edit form |
| PUT | /tasks/{task} | update | Update task |
| DELETE | /tasks/{task} | destroy | Delete task |
| PATCH | /tasks/{task}/toggle | toggle | Toggle completion |

## Creating the Task Controller

Generate a resource controller:

```bash
php artisan make:controller TaskController --resource
```

This creates a controller with all CRUD methods. Edit `app/Http/Controllers/TaskController.php`:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Task;
use App\Models\Category;
use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\View\View;
use Illuminate\Support\Facades\Auth;

class TaskController extends Controller
{
    /**
     * Display a listing of tasks.
     */
    public function index(Request $request): View
    {
        $query = Auth::user()->tasks()->with('category');
        
        // Filter by completion status
        if ($request->has('status')) {
            if ($request->status === 'completed') {
                $query->completed();
            } elseif ($request->status === 'incomplete') {
                $query->incomplete();
            }
        }
        
        // Filter by category
        if ($request->filled('category')) {
            $query->where('category_id', $request->category);
        }
        
        // Filter by priority
        if ($request->filled('priority')) {
            $query->where('priority', $request->priority);
        }
        
        $tasks = $query->latest()->paginate(10);
        $categories = Auth::user()->categories;
        
        return view('tasks.index', [
            'tasks' => $tasks,
            'categories' => $categories,
        ]);
    }

    /**
     * Show the form for creating a new task.
     */
    public function create(): View
    {
        $categories = Auth::user()->categories;
        
        return view('tasks.create', [
            'categories' => $categories,
        ]);
    }

    /**
     * Store a newly created task.
     */
    public function store(Request $request): RedirectResponse
    {
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'category_id' => 'nullable|exists:categories,id',
            'due_date' => 'nullable|date|after_or_equal:today',
            'priority' => 'required|in:low,medium,high',
        ]);
        
        // Verify the category belongs to the user
        if (isset($validated['category_id'])) {
            $category = Category::find($validated['category_id']);
            if ($category->user_id !== Auth::id()) {
                abort(403);
            }
        }
        
        Auth::user()->tasks()->create($validated);
        
        return redirect()->route('tasks.index')
            ->with('success', 'Task created successfully!');
    }

    /**
     * Display the specified task.
     */
    public function show(Task $task): View
    {
        // Ensure user owns this task
        $this->authorize('view', $task);
        
        return view('tasks.show', [
            'task' => $task->load('category'),
        ]);
    }

    /**
     * Show the form for editing the task.
     */
    public function edit(Task $task): View
    {
        $this->authorize('update', $task);
        
        $categories = Auth::user()->categories;
        
        return view('tasks.edit', [
            'task' => $task,
            'categories' => $categories,
        ]);
    }

    /**
     * Update the specified task.
     */
    public function update(Request $request, Task $task): RedirectResponse
    {
        $this->authorize('update', $task);
        
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'category_id' => 'nullable|exists:categories,id',
            'due_date' => 'nullable|date',
            'priority' => 'required|in:low,medium,high',
        ]);
        
        $task->update($validated);
        
        return redirect()->route('tasks.index')
            ->with('success', 'Task updated successfully!');
    }

    /**
     * Remove the specified task.
     */
    public function destroy(Task $task): RedirectResponse
    {
        $this->authorize('delete', $task);
        
        $task->delete();
        
        return redirect()->route('tasks.index')
            ->with('success', 'Task deleted successfully!');
    }

    /**
     * Toggle task completion status.
     */
    public function toggle(Task $task): RedirectResponse
    {
        $this->authorize('update', $task);
        
        $task->update([
            'completed' => !$task->completed,
        ]);
        
        $status = $task->completed ? 'completed' : 'marked as incomplete';
        
        return back()->with('success', "Task {$status}!");
    }
}
```

## Creating the Task Policy

For authorization, create a policy:

```bash
php artisan make:policy TaskPolicy --model=Task
```

Edit `app/Policies/TaskPolicy.php`:

```php
<?php

namespace App\Policies;

use App\Models\Task;
use App\Models\User;

class TaskPolicy
{
    /**
     * Determine if the user can view the task.
     */
    public function view(User $user, Task $task): bool
    {
        return $user->id === $task->user_id;
    }

    /**
     * Determine if the user can update the task.
     */
    public function update(User $user, Task $task): bool
    {
        return $user->id === $task->user_id;
    }

    /**
     * Determine if the user can delete the task.
     */
    public function delete(User $user, Task $task): bool
    {
        return $user->id === $task->user_id;
    }
}
```

## Defining the Routes

Edit `routes/web.php`:

```php
<?php

use App\Http\Controllers\TaskController;
use App\Http\Controllers\CategoryController;
use App\Http\Controllers\DashboardController;
use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('welcome');
});

// Authenticated routes
Route::middleware(['auth', 'verified'])->group(function () {
    // Dashboard
    Route::get('/dashboard', [DashboardController::class, 'index'])
        ->name('dashboard');
    
    // Tasks
    Route::resource('tasks', TaskController::class);
    Route::patch('/tasks/{task}/toggle', [TaskController::class, 'toggle'])
        ->name('tasks.toggle');
    
    // Categories
    Route::resource('categories', CategoryController::class);
});

// Include Breeze auth routes
require __DIR__.'/auth.php';
```

## Creating the Dashboard Controller

```bash
php artisan make:controller DashboardController
```

Edit `app/Http/Controllers/DashboardController.php`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\View\View;
use Illuminate\Support\Facades\Auth;

class DashboardController extends Controller
{
    public function index(): View
    {
        $user = Auth::user();
        
        $stats = [
            'total' => $user->tasks()->count(),
            'completed' => $user->tasks()->completed()->count(),
            'incomplete' => $user->tasks()->incomplete()->count(),
            'overdue' => $user->tasks()->overdue()->count(),
            'due_today' => $user->tasks()->dueToday()->count(),
        ];
        
        $recentTasks = $user->tasks()
            ->with('category')
            ->incomplete()
            ->latest()
            ->take(5)
            ->get();
        
        $upcomingTasks = $user->tasks()
            ->with('category')
            ->incomplete()
            ->whereNotNull('due_date')
            ->where('due_date', '>=', today())
            ->orderBy('due_date')
            ->take(5)
            ->get();
        
        return view('dashboard', [
            'stats' => $stats,
            'recentTasks' => $recentTasks,
            'upcomingTasks' => $upcomingTasks,
        ]);
    }
}
```

## Listing Routes

Verify your routes:

```bash
php artisan route:list --path=tasks
```

```
  GET|HEAD   tasks ............... tasks.index › TaskController@index
  POST       tasks ............... tasks.store › TaskController@store
  GET|HEAD   tasks/create ........ tasks.create › TaskController@create
  GET|HEAD   tasks/{task} ........ tasks.show › TaskController@show
  PUT|PATCH  tasks/{task} ........ tasks.update › TaskController@update
  DELETE     tasks/{task} ........ tasks.destroy › TaskController@destroy
  PATCH      tasks/{task}/toggle . tasks.toggle › TaskController@toggle
```

Now let's create the views!

## Resources

- [Controllers](https://laravel.com/docs/12.x/controllers) — Official documentation on Laravel controllers
- [Routing](https://laravel.com/docs/12.x/routing) — Complete routing documentation

---

> 📘 *This lesson is part of the [Laravel Foundations](https://stanza.dev/courses/laravel-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*