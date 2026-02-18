---
source_course: "laravel-blade-views"
source_lesson: "laravel-blade-views-blade-components"
---

# Blade Components

Blade components are reusable UI elements with their own templates and logic. They're the modern way to build layouts and reusable pieces in Laravel.

## Creating Components

### Anonymous Components (View Only)

Create a file in `resources/views/components/`:

```blade
<!-- resources/views/components/alert.blade.php -->
<div class="alert alert-{{ $type ?? 'info' }}">
    {{ $slot }}
</div>
```

Usage:

```blade
<x-alert type="success">
    Your profile has been updated!
</x-alert>
```

### Class-Based Components

```bash
php artisan make:component Alert
```

Creates two files:
- `app/View/Components/Alert.php` (logic)
- `resources/views/components/alert.blade.php` (template)

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;
use Illuminate\View\View;

class Alert extends Component
{
    public function __construct(
        public string $type = 'info',
        public bool $dismissible = false
    ) {}

    public function render(): View
    {
        return view('components.alert');
    }

    /**
     * Get the CSS class for the alert type.
     */
    public function typeClass(): string
    {
        return match($this->type) {
            'success' => 'bg-green-100 text-green-800',
            'error' => 'bg-red-100 text-red-800',
            'warning' => 'bg-yellow-100 text-yellow-800',
            default => 'bg-blue-100 text-blue-800',
        };
    }
}
```

```blade
<!-- resources/views/components/alert.blade.php -->
<div {{ $attributes->merge(['class' => 'p-4 rounded ' . $typeClass()]) }}>
    {{ $slot }}

    @if ($dismissible)
        <button onclick="this.parentElement.remove()">×</button>
    @endif
</div>
```

## Passing Data to Components

### Attributes (Props)

```blade
{{-- String values --}}
<x-alert type="error">
    Something went wrong!
</x-alert>

{{-- PHP expressions with : prefix --}}
<x-alert :type="$alertType" :dismissible="$canDismiss">
    {{ $message }}
</x-alert>

{{-- Boolean shorthand --}}
<x-alert dismissible>
    Can be closed
</x-alert>
```

### Slots

```blade
{{-- Default slot --}}
<x-card>
    <p>This goes in the default slot.</p>
</x-card>

{{-- Named slots --}}
<x-card>
    <x-slot:header>
        <h2>Card Title</h2>
    </x-slot>

    <p>Card body content.</p>

    <x-slot:footer>
        <button>Action</button>
    </x-slot>
</x-card>
```

```blade
<!-- resources/views/components/card.blade.php -->
<div class="card">
    @isset($header)
        <div class="card-header">{{ $header }}</div>
    @endisset

    <div class="card-body">{{ $slot }}</div>

    @isset($footer)
        <div class="card-footer">{{ $footer }}</div>
    @endisset
</div>
```

## Attribute Merging

```blade
<!-- Component -->
<div {{ $attributes->merge(['class' => 'alert']) }}>
    {{ $slot }}
</div>

<!-- Usage -->
<x-alert class="mb-4" id="main-alert">
    Content
</x-alert>

<!-- Output -->
<div class="alert mb-4" id="main-alert">
    Content
</div>
```

### Conditional Classes

```blade
<div {{ $attributes->class(['alert', 'alert-'.$type, 'font-bold' => $important]) }}>
    {{ $slot }}
</div>
```

### Filtering Attributes

```blade
{{-- Only specific attributes --}}
{{ $attributes->only(['class', 'id']) }}

{{-- All except --}}
{{ $attributes->except(['class']) }}

{{-- Check if exists --}}
@if ($attributes->has('required'))
    <span class="required">*</span>
@endif

{{-- Get specific value --}}
{{ $attributes->get('class', 'default-class') }}
```

## Component Layout

Use components instead of @extends:

```blade
<!-- resources/views/components/layout.blade.php -->
<!DOCTYPE html>
<html>
<head>
    <title>{{ $title ?? 'My App' }}</title>
    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
<body>
    <nav>@include('partials.nav')</nav>

    <main>{{ $slot }}</main>

    <footer>@include('partials.footer')</footer>
</body>
</html>
```

```blade
<!-- resources/views/posts/index.blade.php -->
<x-layout>
    <x-slot:title>All Posts</x-slot>

    <h1>All Posts</h1>

    @foreach ($posts as $post)
        <x-post-card :post="$post" />
    @endforeach
</x-layout>
```

## Nested Components

```bash
php artisan make:component Forms/Input
```

Creates `app/View/Components/Forms/Input.php` and `resources/views/components/forms/input.blade.php`.

```blade
{{-- Usage with dot notation --}}
<x-forms.input name="email" type="email" />
```

## Inline Components

For simple components without a separate view:

```php
class ColorPicker extends Component
{
    public function __construct(
        public string $color = '#000000'
    ) {}

    public function render(): string
    {
        return <<<'blade'
            <div>
                <input type="color" {{ $attributes->merge(['value' => $color]) }}>
            </div>
        blade;
    }
}
```

## Dynamic Components

Render components dynamically:

```blade
<x-dynamic-component :component="$componentName" :message="$message" />

{{-- Equivalent to --}}
@php
    $component = 'x-' . $componentName;
@endphp
<{{ $component }} :message="$message" />
```

## Resources

- [Blade Components](https://laravel.com/docs/12.x/blade#components) — Official Blade components documentation

---

> 📘 *This lesson is part of the [Laravel Blade Templating](https://stanza.dev/courses/laravel-blade-views) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*