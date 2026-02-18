---
source_course: "laravel-blade-views"
source_lesson: "laravel-blade-views-template-inheritance"
---

# Template Inheritance with @extends

Blade's template inheritance lets you define a master layout that child views can extend. This eliminates duplication and ensures consistency.

## Creating a Layout

```blade
<!-- resources/views/layouts/app.blade.php -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@yield('title', 'My App')</title>

    @vite(['resources/css/app.css', 'resources/js/app.js'])

    @stack('styles')
</head>
<body>
    <nav class="navbar">
        @include('partials.navigation')
    </nav>

    <main class="container">
        @yield('content')
    </main>

    <footer>
        @include('partials.footer')
    </footer>

    @stack('scripts')
</body>
</html>
```

## Extending a Layout

```blade
<!-- resources/views/posts/index.blade.php -->
@extends('layouts.app')

@section('title', 'All Posts')

@section('content')
    <h1>All Posts</h1>

    @foreach ($posts as $post)
        <article>
            <h2>{{ $post->title }}</h2>
            <p>{{ $post->excerpt }}</p>
        </article>
    @endforeach
@endsection
```

## @yield and @section

### @yield

Define a placeholder in the layout:

```blade
{{-- Simple yield --}}
@yield('content')

{{-- With default value --}}
@yield('title', 'Default Title')

{{-- With default content block --}}
@yield('sidebar', View::make('partials.default-sidebar'))
```

### @section

Fill the placeholder in child views:

```blade
{{-- Single line --}}
@section('title', 'Page Title')

{{-- Multi-line content --}}
@section('content')
    <h1>Welcome</h1>
    <p>Content here...</p>
@endsection
```

## @section with @show vs @endsection

```blade
{{-- In layout: Define section with default content --}}
@section('sidebar')
    <p>This is the default sidebar.</p>
@show  {{-- Use @show to display immediately --}}

{{-- In child: Can override OR extend --}}
@section('sidebar')
    <p>Custom sidebar content</p>
@endsection
```

### Extending Parent Sections with @parent

```blade
{{-- Layout --}}
@section('sidebar')
    <div class="sidebar-header">Menu</div>
@show

{{-- Child view --}}
@section('sidebar')
    @parent  {{-- Include parent's content --}}
    <nav>Custom navigation</nav>
@endsection

{{-- Result: --}}
<div class="sidebar-header">Menu</div>
<nav>Custom navigation</nav>
```

## @include for Partials

Include reusable view fragments:

```blade
{{-- Basic include --}}
@include('partials.navigation')

{{-- Pass data to the partial --}}
@include('partials.user-card', ['user' => $user])

{{-- Include if exists --}}
@includeIf('partials.optional')

{{-- Include when condition is true --}}
@includeWhen($user->isAdmin, 'partials.admin-menu')

{{-- Include unless condition is true --}}
@includeUnless($user->isGuest, 'partials.user-sidebar')

{{-- Include first existing view --}}
@includeFirst(['custom.header', 'partials.header'])
```

## @each for Collections

Render a view for each item in a collection:

```blade
{{-- Basic each --}}
@each('partials.post-card', $posts, 'post')

{{-- With empty state --}}
@each('partials.post-card', $posts, 'post', 'partials.no-posts')
```

Equivalent to:

```blade
@forelse ($posts as $post)
    @include('partials.post-card', ['post' => $post])
@empty
    @include('partials.no-posts')
@endforelse
```

## Stacks for Scripts and Styles

Push content to named stacks:

```blade
{{-- Layout --}}
<head>
    <link href="/css/app.css" rel="stylesheet">
    @stack('styles')
</head>
<body>
    @yield('content')

    <script src="/js/app.js"></script>
    @stack('scripts')
</body>

{{-- Child view --}}
@push('styles')
    <link href="/css/posts.css" rel="stylesheet">
@endpush

@section('content')
    <h1>Posts</h1>
@endsection

@push('scripts')
    <script src="/js/posts.js"></script>
@endpush

{{-- Prepend to stack --}}
@prepend('scripts')
    <script>var postId = {{ $post->id }};</script>
@endprepend
```

## Complete Layout Example

```blade
<!-- resources/views/layouts/app.blade.php -->
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>@yield('title') - {{ config('app.name') }}</title>

    @vite(['resources/css/app.css', 'resources/js/app.js'])
    @stack('styles')
</head>
<body class="@yield('body-class')">
    <header>
        @include('partials.navigation')
    </header>

    @hasSection('hero')
        <section class="hero">
            @yield('hero')
        </section>
    @endif

    <main class="container mx-auto py-8">
        @if (session('success'))
            <div class="alert alert-success">
                {{ session('success') }}
            </div>
        @endif

        @yield('content')
    </main>

    <footer>
        @include('partials.footer')
    </footer>

    @stack('scripts')
</body>
</html>
```

## Resources

- [Blade Layouts](https://laravel.com/docs/12.x/blade#layouts-using-template-inheritance) — Official documentation on template inheritance

---

> 📘 *This lesson is part of the [Laravel Blade Templating](https://stanza.dev/courses/laravel-blade-views) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*