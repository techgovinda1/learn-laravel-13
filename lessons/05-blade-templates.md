# Lesson 05: Blade Templates and Layouts

**Course:** Learn Laravel 13 – Practical Beginner Course  
**Level:** Beginner  
**Duration:** 40–50 minutes  
**Project:** Task Management System

---

## 1. Learning Objectives

By the end of this lesson, you will be able to:

- Understand Laravel Blade templates.
- Display dynamic data in Blade.
- Use conditional statements and loops.
- Create reusable layouts.
- Use Blade template inheritance.
- Create reusable Blade components.
- Organize views for a maintainable application.

---

## 2. What is Blade?

Blade is Laravel's built-in templating engine.

It allows you to write HTML with dynamic PHP expressions and convenient directives.

Blade files use the `.blade.php` extension and are stored inside:

```text
resources/views/
```

Example:

```php
<h1>Welcome to Laravel</h1>

<p>Today is {{ date('Y-m-d') }}</p>
```

Laravel processes the Blade file and returns normal HTML to the browser.

### Displaying variables

In a controller:

```php
public function index()
{
    $name = 'Laravel Student';

    return view('welcome', compact('name'));
}
```

In `resources/views/welcome.blade.php`:

```html
<h1>Welcome, {{ $name }}</h1>
```

Output:

```text
Welcome, Laravel Student
```

The `{{ }}` syntax escapes output by default, helping protect against cross-site scripting (XSS).

---

## 3. Displaying Data in Blade

Let's continue working with the Task Management System.

Open:

```text
app/Http/Controllers/TaskController.php
```

Make sure your `index()` method passes task data to the view:

```php
public function index()
{
    $tasks = [
        [
            'id' => 1,
            'title' => 'Learn Laravel Routing',
            'status' => 'Completed'
        ],
        [
            'id' => 2,
            'title' => 'Understand Controllers',
            'status' => 'In Progress'
        ],
        [
            'id' => 3,
            'title' => 'Learn Blade Templates',
            'status' => 'Pending'
        ],
    ];

    return view('tasks.index', compact('tasks'));
}
```

Open:

```text
resources/views/tasks/index.blade.php
```

Display the data:

```html
<h1>My Tasks</h1>

@foreach ($tasks as $task)
    <div>
        <h3>{{ $task['title'] }}</h3>
        <p>Status: {{ $task['status'] }}</p>
    </div>
@endforeach
```

The `@foreach` directive loops through the task array.

---

## 4. Conditional Statements

Blade supports conditional directives such as:

- `@if`
- `@elseif`
- `@else`
- `@unless`
- `@isset`
- `@empty`

Example:

```php
@if ($task['status'] === 'Completed')
    <span>Task Completed</span>
@elseif ($task['status'] === 'In Progress')
    <span>Task In Progress</span>
@else
    <span>Task Pending</span>
@endif
```

### Practical exercise

Update your task list:

```html
<h1>My Tasks</h1>

@foreach ($tasks as $task)

    <div>
        <h3>{{ $task['title'] }}</h3>

        @if ($task['status'] === 'Completed')
            <p>Completed</p>
        @elseif ($task['status'] === 'In Progress')
            <p>Currently working</p>
        @else
            <p>Not started</p>
        @endif
    </div>

@endforeach
```

The displayed message changes depending on the task status.

---

## 5. Other Useful Blade Directives

### @forelse

Use `@forelse` when a collection might be empty.

```php
@forelse ($tasks as $task)

    <p>{{ $task['title'] }}</p>

@empty

    <p>No tasks found.</p>

@endforelse
```

### @isset

Check whether a variable exists and is not null.

```php
@isset($name)
    <p>Hello, {{ $name }}</p>
@endisset
```

### @unless

Run content when a condition is false.

```php
@unless ($tasks)
    <p>No task data available.</p>
@endunless
```

---

## 6. Understanding Blade Layouts

Imagine that your application has several pages:

- Task List
- Create Task
- Edit Task
- Task Details

Each page may need the same:

- Header
- Navigation
- Footer
- CSS files

Copying the same HTML into every page creates unnecessary duplication.

Blade layouts solve this problem by allowing multiple pages to share one common structure.

### Step 1: Create a layouts directory

Create:

```text
resources/views/layouts/
```

Inside it, create:

```text
app.blade.php
```

Add:

```html
<!DOCTYPE html>
<html>
<head>
    <title>@yield('title', 'Task Manager')</title>
</head>
<body>

    <header>
        <h1>Task Management System</h1>

        <nav>
            <a href="{{ route('tasks.index') }}">Tasks</a>
            <a href="{{ route('tasks.create') }}">Create Task</a>
        </nav>
    </header>

    <main>
        @yield('content')
    </main>

    <footer>
        <p>Task Manager - Laravel 13</p>
    </footer>

</body>
</html>
```

Important Blade directives:

| Directive | Purpose |
|---|---|
| `@yield('title')` | Defines a section that child views can fill |
| `@yield('content')` | Displays page-specific content |
| `@section` | Defines content in a child view |
| `@endsection` | Ends a section |
| `@extends` | Uses a parent layout |

---

## 7. Use the Layout in Your Task List

Open:

```text
resources/views/tasks/index.blade.php
```

Replace its contents with:

```php
@extends('layouts.app')

@section('title', 'My Tasks')

@section('content')

    <h2>My Tasks</h2>

    @forelse ($tasks as $task)

        <div>
            <h3>{{ $task['title'] }}</h3>

            <p>Status: {{ $task['status'] }}</p>

            <a href="{{ route('tasks.show', $task['id']) }}">
                View Task
            </a>
        </div>

        <hr>

    @empty

        <p>No tasks available.</p>

    @endforelse

@endsection
```

Now the task list automatically uses the common layout.

---

## 8. Use the Same Layout for the Create Task Page

Open:

```text
resources/views/tasks/create.blade.php
```

Replace its contents with:

```php
@extends('layouts.app')

@section('title', 'Create Task')

@section('content')

    <h2>Create a New Task</h2>

    <form action="{{ route('tasks.store') }}" method="POST">

        @csrf

        <label for="title">Task Title:</label>

        <input
            type="text"
            id="title"
            name="title"
            placeholder="Enter task title"
            required
        >

        <button type="submit">Submit Task</button>

    </form>

@endsection
```

Both pages now share the same header, navigation, and footer.

---

## 9. Create a Reusable Blade Component

Blade components allow you to reuse small sections of UI.

For example, each task could be displayed using the same task card.

### Step 1: Create a component

Create:

```text
resources/views/components/task-card.blade.php
```

Add:

```html
<div>
    <h3>{{ $title }}</h3>
    <p>Status: {{ $status }}</p>
</div>

<hr>
```

### Step 2: Use the component

Open:

```text
resources/views/tasks/index.blade.php
```

Inside the `@foreach` loop, use:

```php
@foreach ($tasks as $task)

    <x-task-card
        :title="$task['title']"
        :status="$task['status']"
    />

@endforeach
```

The `:` before an attribute tells Blade to evaluate the value as a PHP expression.

Without `:`, the value is treated as a literal string.

### Why use components?

- Avoid repeating HTML.
- Keep templates organized.
- Reuse the same design across pages.
- Make future UI changes easier.

---

## 10. Add Basic Styling

Let's improve the appearance of the Task Management System.

Create:

```text
public/css/style.css
```

Add:

```css
body {
    font-family: Arial, sans-serif;
    margin: 30px;
    background: #f5f5f5;
}

header,
footer {
    background: #222;
    color: white;
    padding: 15px;
}

nav a {
    color: white;
    margin-right: 15px;
    text-decoration: none;
}

main {
    background: white;
    padding: 20px;
    margin-top: 20px;
}

button {
    padding: 8px 15px;
    cursor: pointer;
}
```

Now include the CSS file inside:

```text
resources/views/layouts/app.blade.php
```

Inside the `<head>` section:

```html
<link rel="stylesheet" href="{{ asset('css/style.css') }}">
```

Laravel's `asset()` helper generates the URL for a public asset.

---

## 11. Test Your Application

Start the Laravel development server:

```bash
php artisan serve
```

Open these pages:

| URL | Expected Result |
|---|---|
| `/tasks` | Task list using the shared layout |
| `/tasks/create` | Create task form using the shared layout |
| `/tasks/1` | Existing task ID response |

Check that:

- The header appears on both pages.
- The navigation links work.
- The footer appears on both pages.
- The task list displays dynamic data.
- The reusable task component works.
- The CSS file loads correctly.

---

## 12. Practical Assignment

Improve the Task Management System using Blade.

### Tasks

1. Create a shared application layout.
2. Add a header, navigation menu, and footer.
3. Update the task list to extend the shared layout.
4. Update the create task page to extend the shared layout.
5. Display task statuses using conditional statements.
6. Use `@forelse` to handle an empty task list.
7. Create a reusable task card component.
8. Add basic CSS styling.

### Expected Project Structure

```text
resources/
└── views/
    ├── layouts/
    │   └── app.blade.php
    ├── components/
    │   └── task-card.blade.php
    └── tasks/
        ├── index.blade.php
        └── create.blade.php

public/
└── css/
    └── style.css
```

---

## 13. Knowledge Check

1. What is Blade in Laravel?
2. What is the file extension for Blade templates?
3. What does `{{ $variable }}` do?
4. What is the difference between `@if` and `@unless`?
5. What is the purpose of `@foreach`?
6. What does `@forelse` do?
7. Why do we use `@extends`?
8. What is the purpose of `@yield`?
9. What is a Blade component?
10. What does Laravel's `asset()` helper do?

---

## 14. Completion Checklist

- [ ] Understand Blade templates.
- [ ] Display dynamic variables.
- [ ] Use conditional statements.
- [ ] Use loops in Blade.
- [ ] Create a shared layout.
- [ ] Use `@extends` and `@section`.
- [ ] Create a reusable Blade component.
- [ ] Load CSS using `asset()`.
- [ ] Test all application pages.

---

## 15. Official Laravel Documentation

- [Blade Templates](https://laravel.com/docs/13.x/blade)
- [Views](https://laravel.com/docs/13.x/views)
- [Assets](https://laravel.com/docs/13.x/vite)

---

## Next Lesson: Lesson 06 – Database and Migrations

In the next lesson, we will learn:

- Database configuration.
- Connecting Laravel to MySQL.
- Understanding migrations.
- Creating database tables.
- Running and rolling back migrations.
- Using Artisan migration commands.

We will prepare the database structure for the Task Management System.