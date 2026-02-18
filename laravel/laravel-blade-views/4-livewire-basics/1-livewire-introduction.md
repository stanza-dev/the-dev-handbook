---
source_course: "laravel-blade-views"
source_lesson: "laravel-blade-views-livewire-introduction"
---

# Introduction to Livewire

Livewire is a full-stack framework for Laravel that makes building dynamic interfaces simple, without leaving the comfort of Laravel and Blade.

## What is Livewire?

Livewire lets you build reactive, dynamic interfaces using PHP instead of JavaScript. Your Blade templates update in real-time through AJAX without writing a single line of JavaScript.

```
┌───────────────────────────────────────────────────────────┐
│                       Browser                              │
├───────────────────────────────────────────────────────────┤
│  ┌─────────────────┐                                      │
│  │ Livewire        │  1. User clicks button               │
│  │ Component       │  ─────────────────────►              │
│  │ (Blade + PHP)   │                                      │
│  │                 │  4. DOM updates automatically        │
│  │                 │  ◄─────────────────────              │
│  └─────────────────┘                                      │
└───────────────────────────────────────────────────────────┘
          │                         ▲
          │ 2. AJAX request         │ 3. Server returns
          │    with action          │    updated HTML
          ▼                         │
┌───────────────────────────────────────────────────────────┐
│                     Laravel Server                         │
│  ┌─────────────────┐                                      │
│  │ Livewire        │                                      │
│  │ Component Class │                                      │
│  │ (PHP)           │                                      │
│  └─────────────────┘                                      │
└───────────────────────────────────────────────────────────┘
```

## Installation

```bash
composer require livewire/livewire
```

Include Livewire scripts in your layout:

```blade
<!DOCTYPE html>
<html>
<head>
    @livewireStyles
</head>
<body>
    {{ $slot }}

    @livewireScripts
</body>
</html>
```

## Creating a Component

```bash
php artisan make:livewire Counter
```

This creates:
- `app/Livewire/Counter.php` - Component class
- `resources/views/livewire/counter.blade.php` - Blade view

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public int $count = 0;

    public function increment(): void
    {
        $this->count++;
    }

    public function decrement(): void
    {
        $this->count--;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

```blade
<!-- resources/views/livewire/counter.blade.php -->
<div>
    <button wire:click="decrement">-</button>
    <span>{{ $count }}</span>
    <button wire:click="increment">+</button>
</div>
```

## Using Components

Include anywhere in Blade:

```blade
<livewire:counter />

{{-- Or with Blade component syntax --}}
@livewire('counter')

{{-- Pass initial values --}}
<livewire:counter :count="5" />
```

## Data Binding

Two-way bind form inputs with `wire:model`:

```php
class SearchUsers extends Component
{
    public string $search = '';
    public array $users = [];

    public function updatedSearch(): void
    {
        $this->users = User::where('name', 'like', '%' . $this->search . '%')
            ->take(10)
            ->get()
            ->toArray();
    }

    public function render()
    {
        return view('livewire.search-users');
    }
}
```

```blade
<div>
    <input type="text" wire:model.live="search" placeholder="Search users...">

    <ul>
        @foreach ($users as $user)
            <li>{{ $user['name'] }}</li>
        @endforeach
    </ul>
</div>
```

### Model Modifiers

```blade
{{-- Update on input (live) --}}
<input wire:model.live="search">

{{-- Debounce live updates --}}
<input wire:model.live.debounce.300ms="search">

{{-- Update on blur --}}
<input wire:model.blur="email">

{{-- Update on change (select, checkbox) --}}
<select wire:model.change="country">
```

## Actions

Call PHP methods from the frontend:

```php
class TodoList extends Component
{
    public array $todos = [];
    public string $newTodo = '';

    public function addTodo(): void
    {
        if (empty($this->newTodo)) return;

        $this->todos[] = [
            'id' => uniqid(),
            'text' => $this->newTodo,
            'completed' => false,
        ];

        $this->newTodo = '';
    }

    public function toggleTodo(string $id): void
    {
        foreach ($this->todos as &$todo) {
            if ($todo['id'] === $id) {
                $todo['completed'] = !$todo['completed'];
            }
        }
    }

    public function deleteTodo(string $id): void
    {
        $this->todos = array_filter(
            $this->todos,
            fn ($todo) => $todo['id'] !== $id
        );
    }

    public function render()
    {
        return view('livewire.todo-list');
    }
}
```

```blade
<div>
    <form wire:submit="addTodo">
        <input type="text" wire:model="newTodo" placeholder="New todo...">
        <button type="submit">Add</button>
    </form>

    <ul>
        @foreach ($todos as $todo)
            <li>
                <input
                    type="checkbox"
                    wire:click="toggleTodo('{{ $todo['id'] }}')"
                    @checked($todo['completed'])
                >
                <span class="{{ $todo['completed'] ? 'line-through' : '' }}">
                    {{ $todo['text'] }}
                </span>
                <button wire:click="deleteTodo('{{ $todo['id'] }}')">×</button>
            </li>
        @endforeach
    </ul>
</div>
```

## Loading States

```blade
<button wire:click="save">
    <span wire:loading.remove>Save</span>
    <span wire:loading>Saving...</span>
</button>

{{-- Target specific action --}}
<span wire:loading wire:target="save">Saving...</span>

{{-- Loading class --}}
<button wire:loading.class="opacity-50" wire:click="save">Save</button>

{{-- Disable while loading --}}
<button wire:loading.attr="disabled" wire:click="save">Save</button>
```

## Resources

- [Livewire Documentation](https://livewire.laravel.com/docs) — Official Livewire documentation

---

> 📘 *This lesson is part of the [Laravel Blade Templating](https://stanza.dev/courses/laravel-blade-views) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*