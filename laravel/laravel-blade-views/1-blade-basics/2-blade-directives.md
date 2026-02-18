---
source_course: "laravel-blade-views"
source_lesson: "laravel-blade-views-blade-directives"
---

# Blade Directives

Blade provides many directives—shortcuts for common PHP control structures that make your templates more readable.

## Conditional Directives

### @if, @elseif, @else, @endif

```blade
@if (count($posts) === 0)
    <p>No posts available.</p>
@elseif (count($posts) === 1)
    <p>One post found!</p>
@else
    <p>{{ count($posts) }} posts found.</p>
@endif
```

### @unless (Inverse of @if)

```blade
@unless (auth()->check())
    <a href="/login">Please log in</a>
@endunless

{{-- Same as: @if (!auth()->check()) --}}
```

### @isset and @empty

```blade
@isset($user)
    <p>Welcome, {{ $user->name }}!</p>
@endisset

@empty($posts)
    <p>No posts to display.</p>
@endempty
```

### @auth and @guest

```blade
@auth
    <p>Welcome back, {{ auth()->user()->name }}!</p>
    <a href="/logout">Logout</a>
@endauth

@guest
    <a href="/login">Login</a>
    <a href="/register">Register</a>
@endguest

{{-- With specific guard --}}
@auth('admin')
    <a href="/admin">Admin Panel</a>
@endauth
```

### @env

```blade
@env('local')
    <div class="debug-bar">Debug Mode</div>
@endenv

@env(['local', 'staging'])
    <div class="warning">Non-production environment</div>
@endenv
```

## Loops

### @foreach

```blade
@foreach ($posts as $post)
    <article>
        <h2>{{ $post->title }}</h2>
        <p>{{ $post->excerpt }}</p>
    </article>
@endforeach
```

### @forelse (Handles Empty Collections)

```blade
@forelse ($posts as $post)
    <article>
        <h2>{{ $post->title }}</h2>
    </article>
@empty
    <p>No posts available.</p>
@endforelse
```

### @for

```blade
@for ($i = 0; $i < 10; $i++)
    <p>Iteration {{ $i }}</p>
@endfor
```

### @while

```blade
@while (true)
    <p>Infinite loop (don't actually do this!)</p>
    @break
@endwhile
```

### @continue and @break

```blade
@foreach ($posts as $post)
    @if ($post->hidden)
        @continue
    @endif

    <h2>{{ $post->title }}</h2>

    @if ($loop->iteration > 10)
        @break
    @endif
@endforeach

{{-- Shorthand with condition --}}
@foreach ($posts as $post)
    @continue($post->hidden)

    <h2>{{ $post->title }}</h2>

    @break($loop->iteration > 10)
@endforeach
```

## The $loop Variable

Inside `@foreach` and `@forelse`, you have access to a special `$loop` variable:

```blade
@foreach ($posts as $post)
    @if ($loop->first)
        <p>First post!</p>
    @endif

    <article class="@if($loop->even) even @endif">
        <span>{{ $loop->iteration }} of {{ $loop->count }}</span>
        <h2>{{ $post->title }}</h2>
    </article>

    @if ($loop->last)
        <p>Last post!</p>
    @endif
@endforeach
```

### Available $loop Properties

| Property | Description |
|----------|-------------|
| `$loop->index` | Current index (0-based) |
| `$loop->iteration` | Current iteration (1-based) |
| `$loop->remaining` | Iterations remaining |
| `$loop->count` | Total items |
| `$loop->first` | Is first iteration? |
| `$loop->last` | Is last iteration? |
| `$loop->even` | Is even iteration? |
| `$loop->odd` | Is odd iteration? |
| `$loop->depth` | Nesting level |
| `$loop->parent` | Parent $loop in nested loops |

### Nested Loops

```blade
@foreach ($categories as $category)
    <h2>{{ $category->name }}</h2>

    @foreach ($category->posts as $post)
        <p>
            Category {{ $loop->parent->iteration }},
            Post {{ $loop->iteration }}
        </p>
    @endforeach
@endforeach
```

## Switch Statements

```blade
@switch($user->role)
    @case('admin')
        <span class="badge badge-red">Admin</span>
        @break

    @case('editor')
        <span class="badge badge-blue">Editor</span>
        @break

    @default
        <span class="badge">User</span>
@endswitch
```

## Class and Style Directives

```blade
{{-- Conditional classes --}}
<div @class([
    'p-4',
    'bg-red-500' => $isError,
    'bg-green-500' => $isSuccess,
    'font-bold' => $isImportant,
])>
    Message
</div>

{{-- Conditional styles --}}
<div @style([
    'background-color: red' => $isError,
    'font-weight: bold' => $isImportant,
])>
    Styled content
</div>
```

## Checked, Selected, Disabled

```blade
<input type="checkbox"
       name="active"
       @checked($user->active) />

<select name="status">
    @foreach ($statuses as $status)
        <option value="{{ $status }}" @selected($status === $user->status)>
            {{ $status }}
        </option>
    @endforeach
</select>

<button @disabled($form->isProcessing)>Submit</button>

<input type="text" @readonly($user->isAdmin) />

<input type="text" @required($isRequired) />
```

## Resources

- [Blade Directives](https://laravel.com/docs/12.x/blade#blade-directives) — Complete list of Blade directives

---

> 📘 *This lesson is part of the [Laravel Blade Templating](https://stanza.dev/courses/laravel-blade-views) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*