---
source_course: "laravel-eloquent-orm"
source_lesson: "laravel-eloquent-orm-what-is-eloquent"
---

# What is Eloquent ORM?

Eloquent is Laravel's **Object-Relational Mapper (ORM)**. It provides a beautiful, simple ActiveRecord implementation for working with your database, where each database table has a corresponding "Model" class.

## What is an ORM?

An ORM maps:
- **PHP classes** → Database tables
- **Class properties** → Table columns
- **Class instances** → Table rows

```
PHP Class: User                    Database Table: users
┌─────────────────────────┐        ┌────┬───────────┬─────────────────┐
│ class User extends Model │        │ id │ name      │ email           │
│ {                        │        ├────┼───────────┼─────────────────┤
│   // Properties mapped   │   ───► │ 1  │ "John"    │ "john@mail.com" │
│   // automatically       │        │ 2  │ "Jane"    │ "jane@mail.com" │
│ }                        │        └────┴───────────┴─────────────────┘
└─────────────────────────┘
```

## Why Use Eloquent?

### Traditional SQL Approach

```php
// Without ORM - raw PDO/SQL
$pdo = new PDO('mysql:host=localhost;dbname=app', 'root', 'password');
$stmt = $pdo->prepare('SELECT * FROM users WHERE id = ?');
$stmt->execute([1]);
$user = $stmt->fetch(PDO::FETCH_ASSOC);

$name = $user['name'];
```

### Eloquent Approach

```php
// With Eloquent - elegant and simple
$user = User::find(1);
$name = $user->name;
```

## Benefits of Eloquent

| Feature | Benefit |
|---------|----------|
| **Expressive Syntax** | Write readable, intuitive database code |
| **Relationships** | Define related data with simple methods |
| **Mass Assignment** | Securely create/update multiple fields |
| **Soft Deletes** | "Delete" records without removing data |
| **Events & Observers** | Hook into model lifecycle events |
| **Query Scopes** | Reusable query constraints |
| **Mutators & Casts** | Transform data automatically |
| **Serialization** | Easy JSON/array conversion |

## Your First Model

A simple Eloquent model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    // That's it! Eloquent handles everything else.
}
```

With this simple class, you can:

```php
// Create
$post = Post::create(['title' => 'Hello World', 'body' => '...']);

// Read
$post = Post::find(1);
$posts = Post::all();
$posts = Post::where('published', true)->get();

// Update
$post->title = 'Updated Title';
$post->save();

// Delete
$post->delete();
```

## Eloquent Conventions

Eloquent uses sensible defaults based on conventions:

### Table Names

```php
class User extends Model {}      // Table: users
class Post extends Model {}      // Table: posts
class Category extends Model {}  // Table: categories (pluralized)
class Person extends Model {}    // Table: people (irregular plural)
```

### Primary Keys

```php
// Default: id column, auto-incrementing integer
$user = User::find(1);  // Finds by 'id' column
```

### Timestamps

```php
// Default: created_at and updated_at columns
// Automatically managed by Eloquent
$post->created_at;  // Carbon instance
$post->updated_at;  // Carbon instance
```

## The Models Directory

Models live in `app/Models/`:

```
app/Models/
├── User.php        # User model
├── Post.php        # Post model
├── Comment.php     # Comment model
└── Category.php    # Category model
```

## Model vs Query Builder

Eloquent is built on top of Laravel's Query Builder:

```php
// Query Builder - returns stdClass objects
DB::table('users')->where('active', true)->get();

// Eloquent - returns User model instances
User::where('active', true)->get();
```

Eloquent models have additional features like:
- Relationships
- Events/observers
- Accessors/mutators
- Serialization options

## Resources

- [Eloquent: Getting Started](https://laravel.com/docs/12.x/eloquent) — Official introduction to Eloquent ORM

---

> 📘 *This lesson is part of the [Laravel Eloquent ORM](https://stanza.dev/courses/laravel-eloquent-orm) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*