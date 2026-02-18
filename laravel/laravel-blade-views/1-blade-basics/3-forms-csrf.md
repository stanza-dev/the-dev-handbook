---
source_course: "laravel-blade-views"
source_lesson: "laravel-blade-views-forms-csrf"
---

# Forms and CSRF Protection

Blade provides helpful directives for working with HTML forms, including CSRF protection and method spoofing.

## CSRF Protection

Laravel automatically generates a CSRF token for each active user session. Include it in forms:

```blade
<form method="POST" action="/posts">
    @csrf

    <input type="text" name="title">
    <button type="submit">Create</button>
</form>
```

`@csrf` generates:

```html
<input type="hidden" name="_token" value="abc123...">
```

**Without `@csrf`**, POST, PUT, PATCH, and DELETE requests will fail with a 419 error.

## Method Spoofing

HTML forms only support GET and POST. For PUT, PATCH, DELETE, use `@method`:

```blade
{{-- Update form --}}
<form method="POST" action="/posts/{{ $post->id }}">
    @csrf
    @method('PUT')

    <input type="text" name="title" value="{{ $post->title }}">
    <button type="submit">Update</button>
</form>

{{-- Delete form --}}
<form method="POST" action="/posts/{{ $post->id }}">
    @csrf
    @method('DELETE')

    <button type="submit">Delete</button>
</form>
```

`@method('PUT')` generates:

```html
<input type="hidden" name="_method" value="PUT">
```

## Displaying Validation Errors

### The @error Directive

```blade
<form method="POST" action="/posts">
    @csrf

    <div>
        <label for="title">Title</label>
        <input type="text"
               name="title"
               id="title"
               value="{{ old('title') }}"
               class="@error('title') is-invalid @enderror">

        @error('title')
            <span class="error">{{ $message }}</span>
        @enderror
    </div>

    <div>
        <label for="body">Body</label>
        <textarea name="body">{{ old('body') }}</textarea>

        @error('body')
            <span class="error">{{ $message }}</span>
        @enderror
    </div>

    <button type="submit">Create Post</button>
</form>
```

### Named Error Bags

For multiple forms on one page:

```blade
{{-- In controller --}}
return back()->withErrors($validator, 'login');

{{-- In Blade --}}
@error('email', 'login')
    <span class="error">{{ $message }}</span>
@enderror
```

### All Errors

```blade
@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

## The old() Helper

Repopulate form fields after validation errors:

```blade
<input type="text"
       name="title"
       value="{{ old('title') }}">

{{-- With default value --}}
<input type="text"
       name="title"
       value="{{ old('title', $post->title) }}">

{{-- For textareas --}}
<textarea name="body">{{ old('body', $post->body) }}</textarea>

{{-- For checkboxes --}}
<input type="checkbox"
       name="published"
       @checked(old('published', $post->published))>

{{-- For select --}}
<select name="category_id">
    @foreach ($categories as $category)
        <option value="{{ $category->id }}"
                @selected(old('category_id', $post->category_id) == $category->id)>
            {{ $category->name }}
        </option>
    @endforeach
</select>
```

## Complete Form Example

```blade
<form method="POST" action="{{ route('posts.store') }}" enctype="multipart/form-data">
    @csrf

    {{-- Show all errors at top --}}
    @if ($errors->any())
        <div class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded mb-4">
            <strong>Whoops!</strong> There were some problems with your input.
            <ul class="mt-2 list-disc list-inside">
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    {{-- Title field --}}
    <div class="mb-4">
        <label for="title" class="block font-medium">Title</label>
        <input type="text"
               name="title"
               id="title"
               value="{{ old('title') }}"
               class="w-full border rounded p-2 @error('title') border-red-500 @enderror">
        @error('title')
            <p class="text-red-500 text-sm mt-1">{{ $message }}</p>
        @enderror
    </div>

    {{-- Category select --}}
    <div class="mb-4">
        <label for="category_id" class="block font-medium">Category</label>
        <select name="category_id" id="category_id" class="w-full border rounded p-2">
            <option value="">Select a category</option>
            @foreach ($categories as $category)
                <option value="{{ $category->id }}" @selected(old('category_id') == $category->id)>
                    {{ $category->name }}
                </option>
            @endforeach
        </select>
        @error('category_id')
            <p class="text-red-500 text-sm mt-1">{{ $message }}</p>
        @enderror
    </div>

    {{-- Body textarea --}}
    <div class="mb-4">
        <label for="body" class="block font-medium">Content</label>
        <textarea name="body"
                  id="body"
                  rows="5"
                  class="w-full border rounded p-2 @error('body') border-red-500 @enderror">{{ old('body') }}</textarea>
        @error('body')
            <p class="text-red-500 text-sm mt-1">{{ $message }}</p>
        @enderror
    </div>

    {{-- Published checkbox --}}
    <div class="mb-4">
        <label class="flex items-center">
            <input type="checkbox" name="published" value="1" @checked(old('published'))>
            <span class="ml-2">Publish immediately</span>
        </label>
    </div>

    {{-- Submit --}}
    <button type="submit" class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600">
        Create Post
    </button>
</form>
```

## Resources

- [CSRF Protection](https://laravel.com/docs/12.x/csrf) — Official CSRF protection documentation

---

> 📘 *This lesson is part of the [Laravel Blade Templating](https://stanza.dev/courses/laravel-blade-views) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*