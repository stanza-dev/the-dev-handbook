---
source_course: "laravel-blade-views"
source_lesson: "laravel-blade-views-practical-components"
---

# Building Practical Components

Let's build real-world components you'll use in every Laravel project.

## Form Input Component

```bash
php artisan make:component Forms/Input
```

```php
<?php

namespace App\View\Components\Forms;

use Illuminate\View\Component;
use Illuminate\View\View;

class Input extends Component
{
    public function __construct(
        public string $name,
        public string $type = 'text',
        public ?string $label = null,
        public ?string $value = null,
        public bool $required = false,
    ) {
        $this->label = $label ?? ucfirst(str_replace('_', ' ', $name));
        $this->value = old($name, $value);
    }

    public function render(): View
    {
        return view('components.forms.input');
    }
}
```

```blade
<!-- resources/views/components/forms/input.blade.php -->
<div class="mb-4">
    <label for="{{ $name }}" class="block text-sm font-medium text-gray-700">
        {{ $label }}
        @if ($required)
            <span class="text-red-500">*</span>
        @endif
    </label>

    <input
        type="{{ $type }}"
        name="{{ $name }}"
        id="{{ $name }}"
        value="{{ $value }}"
        {{ $attributes->merge([
            'class' => 'mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500'
                . ($errors->has($name) ? ' border-red-500' : '')
        ]) }}
        @if($required) required @endif
    >

    @error($name)
        <p class="mt-1 text-sm text-red-600">{{ $message }}</p>
    @enderror
</div>
```

Usage:

```blade
<x-forms.input name="email" type="email" required />
<x-forms.input name="phone" label="Phone Number" />
<x-forms.input name="title" :value="$post->title" />
```

## Button Component

```blade
<!-- resources/views/components/button.blade.php -->
@props([
    'type' => 'button',
    'variant' => 'primary',
    'size' => 'md',
])

@php
$variants = [
    'primary' => 'bg-blue-600 hover:bg-blue-700 text-white',
    'secondary' => 'bg-gray-200 hover:bg-gray-300 text-gray-800',
    'danger' => 'bg-red-600 hover:bg-red-700 text-white',
    'success' => 'bg-green-600 hover:bg-green-700 text-white',
];

$sizes = [
    'sm' => 'px-3 py-1.5 text-sm',
    'md' => 'px-4 py-2',
    'lg' => 'px-6 py-3 text-lg',
];
@endphp

<button
    type="{{ $type }}"
    {{ $attributes->merge([
        'class' => 'rounded font-medium transition-colors ' . $variants[$variant] . ' ' . $sizes[$size]
    ]) }}
>
    {{ $slot }}
</button>
```

Usage:

```blade
<x-button>Click Me</x-button>
<x-button type="submit" variant="success">Save</x-button>
<x-button variant="danger" size="sm" onclick="confirm('Sure?')">Delete</x-button>
```

## Modal Component

```blade
<!-- resources/views/components/modal.blade.php -->
@props([
    'name',
    'title' => '',
    'maxWidth' => 'md',
])

@php
$maxWidthClasses = [
    'sm' => 'max-w-sm',
    'md' => 'max-w-md',
    'lg' => 'max-w-lg',
    'xl' => 'max-w-xl',
    '2xl' => 'max-w-2xl',
];
@endphp

<div
    x-data="{ open: false }"
    x-on:open-modal.window="$event.detail === '{{ $name }}' ? open = true : null"
    x-on:close-modal.window="$event.detail === '{{ $name }}' ? open = false : null"
    x-on:keydown.escape.window="open = false"
    x-show="open"
    class="fixed inset-0 z-50 overflow-y-auto"
    style="display: none;"
>
    <!-- Backdrop -->
    <div
        x-show="open"
        x-transition:enter="ease-out duration-300"
        x-transition:enter-start="opacity-0"
        x-transition:enter-end="opacity-100"
        class="fixed inset-0 bg-gray-500 bg-opacity-75"
        @click="open = false"
    ></div>

    <!-- Modal -->
    <div class="flex min-h-screen items-center justify-center p-4">
        <div
            x-show="open"
            x-transition:enter="ease-out duration-300"
            x-transition:enter-start="opacity-0 scale-95"
            x-transition:enter-end="opacity-100 scale-100"
            class="relative bg-white rounded-lg shadow-xl {{ $maxWidthClasses[$maxWidth] }} w-full"
        >
            @if ($title)
                <div class="px-6 py-4 border-b">
                    <h3 class="text-lg font-semibold">{{ $title }}</h3>
                </div>
            @endif

            <div class="px-6 py-4">
                {{ $slot }}
            </div>

            @isset($footer)
                <div class="px-6 py-4 border-t bg-gray-50 flex justify-end gap-2">
                    {{ $footer }}
                </div>
            @endisset
        </div>
    </div>
</div>
```

Usage:

```blade
<x-button @click="$dispatch('open-modal', 'confirm-delete')">
    Delete
</x-button>

<x-modal name="confirm-delete" title="Confirm Deletion">
    <p>Are you sure you want to delete this item?</p>

    <x-slot:footer>
        <x-button variant="secondary" @click="$dispatch('close-modal', 'confirm-delete')">
            Cancel
        </x-button>
        <x-button variant="danger" type="submit" form="delete-form">
            Delete
        </x-button>
    </x-slot>
</x-modal>
```

## Card Component

```blade
<!-- resources/views/components/card.blade.php -->
@props(['padding' => true])

<div {{ $attributes->merge(['class' => 'bg-white rounded-lg shadow']) }}>
    @isset($header)
        <div class="px-6 py-4 border-b font-semibold">
            {{ $header }}
        </div>
    @endisset

    <div @class(['px-6 py-4' => $padding])>
        {{ $slot }}
    </div>

    @isset($footer)
        <div class="px-6 py-4 border-t bg-gray-50">
            {{ $footer }}
        </div>
    @endisset
</div>
```

## Complete Form Example

```blade
<x-layout>
    <x-slot:title>Create Post</x-slot>

    <x-card class="max-w-2xl mx-auto">
        <x-slot:header>Create New Post</x-slot>

        <form method="POST" action="{{ route('posts.store') }}">
            @csrf

            <x-forms.input name="title" required />
            <x-forms.input name="slug" />

            <x-forms.textarea name="body" rows="10" required />

            <x-forms.select name="category_id" :options="$categories" required />

            <x-forms.checkbox name="published" label="Publish immediately" />
        </form>

        <x-slot:footer>
            <x-button variant="secondary" href="{{ route('posts.index') }}">
                Cancel
            </x-button>
            <x-button type="submit" form="create-post-form">
                Create Post
            </x-button>
        </x-slot>
    </x-card>
</x-layout>
```

## Resources

- [Blade Components](https://laravel.com/docs/12.x/blade#components) — Complete Blade components documentation

---

> 📘 *This lesson is part of the [Laravel Blade Templating](https://stanza.dev/courses/laravel-blade-views) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*