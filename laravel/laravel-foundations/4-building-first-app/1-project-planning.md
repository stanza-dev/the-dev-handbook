---
source_course: "laravel-foundations"
source_lesson: "laravel-foundations-project-planning"
---

# Planning Your Application

Before writing any code, let's plan the task management application we'll build. Good planning saves time and helps you understand how Laravel's pieces fit together.

## What We're Building

We'll create a **Task Manager** application with these features:

- User registration and authentication
- Create, read, update, and delete tasks
- Mark tasks as complete/incomplete
- Organize tasks by categories
- Filter and search tasks
- Dashboard with task statistics

## Application Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Task Manager                             │
├─────────────────────────────────────────────────────────────────┤
│  Routes          │  Controllers      │  Views                   │
│  ─────────────   │  ─────────────    │  ─────────────          │
│  /               │  HomeController   │  home.blade.php          │
│  /dashboard      │  DashboardCtrl    │  dashboard.blade.php     │
│  /tasks          │  TaskController   │  tasks/index.blade.php   │
│  /tasks/create   │  TaskController   │  tasks/create.blade.php  │
│  /tasks/{id}     │  TaskController   │  tasks/show.blade.php    │
│  /tasks/{id}/edit│  TaskController   │  tasks/edit.blade.php    │
│  /categories     │  CategoryCtrl     │  categories/*.blade.php  │
├─────────────────────────────────────────────────────────────────┤
│  Models                                                          │
│  ─────────────────────────────────────────────────────────────  │
│  User        │  Task           │  Category                       │
│  - id        │  - id           │  - id                           │
│  - name      │  - user_id (FK) │  - user_id (FK)                 │
│  - email     │  - category_id  │  - name                         │
│  - password  │  - title        │  - color                        │
│              │  - description  │  - created_at                   │
│              │  - completed    │  - updated_at                   │
│              │  - due_date     │                                 │
│              │  - created_at   │                                 │
│              │  - updated_at   │                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Database Relationships

```
User (1) ──────< (Many) Task
  │
  └──────< (Many) Category

Category (1) ──────< (Many) Task
```

- A **User** has many **Tasks**
- A **User** has many **Categories**
- A **Category** has many **Tasks**
- A **Task** belongs to a **User** and a **Category**

## User Stories

Let's define what users can do:

1. **As a visitor**, I can register for an account
2. **As a visitor**, I can log in to my account
3. **As a user**, I can create new tasks with title, description, and due date
4. **As a user**, I can view all my tasks
5. **As a user**, I can mark tasks as complete or incomplete
6. **As a user**, I can edit my tasks
7. **As a user**, I can delete my tasks
8. **As a user**, I can create categories to organize my tasks
9. **As a user**, I can assign tasks to categories
10. **As a user**, I can filter tasks by category or completion status
11. **As a user**, I can see statistics on my dashboard

## Technology Stack

| Layer | Technology | Purpose |
|-------|------------|----------|
| Backend | Laravel 12 | Framework |
| Database | SQLite | Development database |
| ORM | Eloquent | Database interactions |
| Templating | Blade | Server-side rendering |
| CSS | Tailwind CSS | Styling |
| Auth | Laravel Breeze | Authentication scaffolding |

## Development Steps

1. **Setup**: Create project, install Breeze for auth
2. **Database**: Create migrations for tasks and categories
3. **Models**: Define Eloquent models and relationships
4. **Routes**: Define web routes for all pages
5. **Controllers**: Create controllers with CRUD methods
6. **Views**: Build Blade templates with Tailwind CSS
7. **Validation**: Add form validation
8. **Authorization**: Ensure users can only access their own data
9. **Polish**: Add filtering, searching, and statistics

## Project Setup

Let's start by creating the project:

```bash
# Create a new Laravel project
laravel new task-manager

# Navigate to the project
cd task-manager

# Install Laravel Breeze for authentication
composer require laravel/breeze --dev

# Install Breeze with Blade (simple setup)
php artisan breeze:install blade

# Install NPM dependencies
npm install

# Run migrations
php artisan migrate

# Start the development server
composer run dev
```

Now visit http://localhost:8000 - you should see the Laravel welcome page with working Register and Login links!

## What Breeze Gives You

Laravel Breeze provides:

- **Registration page** at `/register`
- **Login page** at `/login`
- **Password reset** flow
- **Email verification** (optional)
- **Profile management** at `/profile`
- **Dashboard** at `/dashboard`
- **Tailwind CSS** styling
- **Alpine.js** for interactivity

## Next Steps

With authentication ready, we'll:

1. Create the database structure
2. Build the Task and Category models
3. Create controllers and views
4. Add the business logic

Let's start building!

## Resources

- [Laravel Breeze](https://laravel.com/docs/12.x/starter-kits#laravel-breeze) — Official documentation for Laravel Breeze authentication starter kit

---

> 📘 *This lesson is part of the [Laravel Foundations](https://stanza.dev/courses/laravel-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*