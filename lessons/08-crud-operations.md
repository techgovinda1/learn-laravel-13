# Lesson 08: CRUD Operations in Laravel

**Course:** Learn Laravel 13 – Practical Beginner Course  
**Level:** Beginner  
**Duration:** 60–75 minutes  
**Project:** Task Management System

---

## 1. Learning Objectives

By the end of this lesson, you will be able to:

- Understand CRUD operations.
- Create resource controllers.
- Use resource routes.
- Create and store database records.
- Display records from MySQL.
- View individual records.
- Edit and update records.
- Delete records.
- Use route model binding.
- Connect forms, controllers, models, and database tables.

---

## 2. What is CRUD?

CRUD represents the four basic operations performed by most applications.

| Operation | Meaning | Task Manager Example |
|---|---|---|
| Create | Add new data | Create a task |
| Read | Retrieve data | Display tasks |
| Update | Modify existing data | Edit a task |
| Delete | Remove data | Delete a task |

In this lesson, we will implement all four operations using Laravel, MySQL, Eloquent, and Blade.

### How CRUD works in Laravel

```text
Browser
   |
   v
Route
   |
   v
Controller
   |
   v
Model (Eloquent)
   |
   v
Database
   |
   v
Controller
   |
   v
Blade View
   |
   v
Browser
```

---

## 3. Create a Resource Controller

Laravel provides resource controllers to organize CRUD functionality.

Run:

```bash
php artisan make:controller TaskController --resource
```

Laravel creates:

```text
app/Http/Controllers/TaskController.php
```

A resource controller includes seven methods:

| Method | Purpose |
|---|---|
| `index()` | Display all tasks |
| `create()` | Show the create form |
| `store()` | Save a new task |
| `show()` | Display one task |
| `edit()` | Show the edit form |
| `update()` | Update an existing task |
| `destroy()` | Delete a task |

These methods follow Laravel's resource controller conventions.

---

## 4. Register Resource Routes

Open:

```text
routes/web.php
```

Remove the previous individual task routes from earlier lessons to avoid duplicate route definitions.

Add:

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\TaskController;

Route::resource('tasks', TaskController::class);
```

This single line registers all seven standard resource routes.

### Understand the generated routes

Run:

```bash
php artisan route:list --path=tasks
```

Laravel registers routes similar to:

| HTTP Method | URL | Controller Method | Route Name |
|---|---|---|---|
| GET | `/tasks` | index | tasks.index |
| GET | `/tasks/create` | create | tasks.create |
| POST | `/tasks` | store | tasks.store |
| GET | `/tasks/{task}` | show | tasks.show |
| GET | `/tasks/{task}/edit` | edit | tasks.edit |
| PUT/PATCH | `/tasks/{task}` | update | tasks.update |
| DELETE | `/tasks/{task}` | destroy | tasks.destroy |

**Important:** The `/tasks/create` route is registered before the dynamic `/tasks/{task}` route so that `create` is not treated as a task identifier.

---

## 5. Understand Route Model Binding

Laravel supports route model binding.

Instead of receiving an ID and manually finding a task:

```php
public function show(string $id)
{
    $task = Task::findOrFail($id);

    return view('tasks.show', compact('task'));
}
```

You can type-hint the model directly:

```php
public function show(Task $task)
{
    return view('tasks.show', compact('task'));
}
```

Laravel automatically finds the task using its ID.

If the task does not exist, Laravel returns a 404 response.

This is called **implicit route model binding**.

Make sure the controller imports the model:

```php
use App\Models\Task;
```

---

## 6. Implement the Controller

Open:

```text
app/Http/Controllers/TaskController.php
```

Replace the generated contents with:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Task;
use Illuminate\Http\Request;

class TaskController extends Controller
{
    // Display all tasks
    public function index()
    {
        $tasks = Task::latest()->get();

        return view('tasks.index', compact('tasks'));
    }

    // Show the create form
    public function create()
    {
        return view('tasks.create');
    }

    // Store a new task
    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'status' => 'required|string',
            'priority' => 'required|string',
            'due_date' => 'nullable|date',
        ]);

        Task::create($validated);

        return redirect()
            ->route('tasks.index')
            ->with('success', 'Task created successfully.');
    }

    // Display one task
    public function show(Task $task)
    {
        return view('tasks.show', compact('task'));
    }

    // Show the edit form
    public function edit(Task $task)
    {
        return view('tasks.edit', compact('task'));
    }

    // Update an existing task
    public function update(Request $request, Task $task)
    {
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'status' => 'required|string',
            'priority' => 'required|string',
            'due_date' => 'nullable|date',
        ]);

        $task->update($validated);

        return redirect()
            ->route('tasks.index')
            ->with('success', 'Task updated successfully.');
    }

    // Delete a task
    public function destroy(Task $task)
    {
        $task->delete();

        return redirect()
            ->route('tasks.index')
            ->with('success', 'Task deleted successfully.');
    }
}
```

### Understanding the controller

| Code | Purpose |
|---|---|
| `Task::latest()->get()` | Retrieves tasks, newest first |
| `Task::create($validated)` | Creates a new database record |
| `Task $task` | Uses route model binding |
| `$task->update()` | Updates an existing record |
| `$task->delete()` | Deletes a record |
| `redirect()->route()` | Redirects to a named route |
| `with('success', ...)` | Adds a flash message to the session |
| `$request->validate()` | Validates submitted form data |

**Note:** Validation is introduced here because forms should not save arbitrary data. We will explore Laravel validation rules in more detail in a later lesson.

---

## 7. Create a Reusable Task Form

Instead of writing the same form twice, we will create one reusable Blade partial for both the create and edit pages.

Create:

```text
resources/views/tasks/
```

Inside it, create:

```text
_form.blade.php
```

Add:

```php
<div>
    <label for="title">Task Title</label>

    <input
        type="text"
        id="title"
        name="title"
        value="{{ old('title', $task?->title) }}"
        required
    >

    @error('title')
        <p>{{ $message }}</p>
    @enderror
</div>

<div>
    <label for="description">Description</label>

    <textarea
        id="description"
        name="description"
    >{{ old('description', $task?->description) }}</textarea>

    @error('description')
        <p>{{ $message }}</p>
    @enderror
</div>

<div>
    <label for="status">Status</label>

    <select id="status" name="status" required>
        <option value="Pending"
            {{ old('status', $task?->status) === 'Pending' ? 'selected' : '' }}>
            Pending
        </option>

        <option value="In Progress"
            {{ old('status', $task?->status) === 'In Progress' ? 'selected' : '' }}>
            In Progress
        </option>

        <option value="Completed"
            {{ old('status', $task?->status) === 'Completed' ? 'selected' : '' }}>
            Completed
        </option>
    </select>

    @error('status')
        <p>{{ $message }}</p>
    @enderror
</div>

<div>
    <label for="priority">Priority</label>

    <select id="priority" name="priority" required>
        <option value="Low"
            {{ old('priority', $task?->priority) === 'Low' ? 'selected' : '' }}>
            Low
        </option>

        <option value="Normal"
            {{ old('priority', $task?->priority) === 'Normal' ? 'selected' : '' }}>
            Normal
        </option>

        <option value="High"
            {{ old('priority', $task?->priority) === 'High' ? 'selected' : '' }}>
            High
        </option>
    </select>

    @error('priority')
        <p>{{ $message }}</p>
    @enderror
</div>

<div>
    <label for="due_date">Due Date</label>

    <input
        type="date"
        id="due_date"
        name="due_date"
        value="{{ old('due_date', $task?->due_date) }}"
    >

    @error('due_date')
        <p>{{ $message }}</p>
    @enderror
</div>

<button type="submit">{{ $buttonText }}</button>
```

The `$task` variable may be `null` when creating a task. The null-safe operator `?->` prevents an error when accessing its properties.

---

## 8. Create the Create Task Page

Create:

```text
resources/views/tasks/create.blade.php
```

Add:

```php
@extends('layouts.app')

@section('title', 'Create Task')

@section('content')

    <h2>Create New Task</h2>

    <form action="{{ route('tasks.store') }}" method="POST">

        @csrf

        @include('tasks._form', [
            'task' => null,
            'buttonText' => 'Create Task'
        ])

    </form>

@endsection
```

### Important concepts

- `route('tasks.store')` generates the URL for the store route.
- `method="POST"` submits the form.
- `@csrf` adds Laravel's CSRF protection token.
- `@include` reuses the shared form partial.

---

## 9. Create the Task List Page

Create or update:

```text
resources/views/tasks/index.blade.php
```

Add:

```php
@extends('layouts.app')

@section('title', 'All Tasks')

@section('content')

    <h2>All Tasks</h2>

    <a href="{{ route('tasks.create') }}">
        Create New Task
    </a>

    @if (session('success'))
        <p>{{ session('success') }}</p>
    @endif

    @forelse ($tasks as $task)

        <div>
            <h3>{{ $task->title }}</h3>

            <p>Status: {{ $task->status }}</p>

            <p>Priority: {{ $task->priority }}</p>

            <a href="{{ route('tasks.show', $task) }}">
                View
            </a>

            <a href="{{ route('tasks.edit', $task) }}">
                Edit
            </a>

            <form
                action="{{ route('tasks.destroy', $task) }}"
                method="POST"
                onsubmit="return confirm('Are you sure you want to delete this task?')"
            >
                @csrf
                @method('DELETE')

                <button type="submit">Delete</button>
            </form>
        </div>

        <hr>

    @empty

        <p>No tasks found.</p>

    @endforelse

@endsection
```

This page displays tasks from the database and provides links for viewing, editing, and deleting them.

---

## 10. Create the Task Details Page

Create:

```text
resources/views/tasks/show.blade.php
```

Add:

```php
@extends('layouts.app')

@section('title', 'Task Details')

@section('content')

    <h2>Task Details</h2>

    <h3>{{ $task->title }}</h3>

    <p>
        Description: {{ $task->description ?? 'No description' }}
    </p>

    <p>Status: {{ $task->status }}</p>

    <p>Priority: {{ $task->priority }}</p>

    <p>Due Date: {{ $task->due_date ?? 'Not specified' }}</p>

    <a href="{{ route('tasks.index') }}">Back to Tasks</a>

    <a href="{{ route('tasks.edit', $task) }}">Edit Task</a>

@endsection
```

Visit `/tasks/1` to view the details of the task with ID 1, assuming that record exists.

---

## 11. Create the Edit Task Page

Create:

```text
resources/views/tasks/edit.blade.php
```

Add:

```php
@extends('layouts.app')

@section('title', 'Edit Task')

@section('content')

    <h2>Edit Task</h2>

    <form action="{{ route('tasks.update', $task) }}" method="POST">

        @csrf
        @method('PUT')

        @include('tasks._form', [
            'task' => $task,
            'buttonText' => 'Update Task'
        ])

    </form>

@endsection
```

### Why do we use `@method('PUT')`?

HTML forms natively support only:

- GET
- POST

Laravel uses method spoofing to support other HTTP methods through forms.

```php
@method('PUT')
```

This tells Laravel to treat the form submission as a PUT request.

The same approach is used for DELETE:

```php
@method('DELETE')
```

---

## 12. Test All CRUD Operations

Start the development server:

```bash
php artisan serve
```

Visit:

```text
http://localhost:8000/tasks
```

Test each operation.

| Operation | URL / Action | Expected Result |
|---|---|---|
| Create | Click Create New Task | Display task form |
| Store | Submit the form | Save task to MySQL |
| Read | Open `/tasks` | Display all tasks |
| Show | Click View | Display individual task |
| Edit | Click Edit | Display existing task data |
| Update | Submit edited form | Update database record |
| Delete | Click Delete | Remove task from database |

Check the records in phpMyAdmin to confirm that changes are saved.

---

## 13. Understand the Complete CRUD Flow

```text
CREATE
/tasks/create
      |
      v
Create Form
      |
      v
POST /tasks
      |
      v
TaskController@store
      |
      v
Task::create()
      |
      v
MySQL


READ
GET /tasks
      |
      v
TaskController@index
      |
      v
Task::latest()->get()
      |
      v
tasks/index.blade.php


UPDATE
/tasks/{task}/edit
      |
      v
Edit Form
      |
      v
PUT /tasks/{task}
      |
      v
TaskController@update
      |
      v
$task->update()


DELETE
DELETE /tasks/{task}
      |
      v
TaskController@destroy
      |
      v
$task->delete()
```

---

## 14. Practical Assignment

Complete the Task Management System.

### Requirements

1. Display all tasks from MySQL.
2. Create a new task using a form.
3. Display individual task details.
4. Edit task information.
5. Update task status and priority.
6. Delete a task.
7. Display success messages after create, update, and delete.
8. Display validation errors.
9. Reuse the same form for creating and editing tasks.
10. Use resource routes and a resource controller.

### Expected Project Structure

```text
app/
├── Http/
│   └── Controllers/
│       └── TaskController.php
└── Models/
    └── Task.php

resources/
└── views/
    ├── layouts/
    │   └── app.blade.php
    └── tasks/
        ├── _form.blade.php
        ├── index.blade.php
        ├── create.blade.php
        ├── show.blade.php
        └── edit.blade.php

routes/
└── web.php
```

---

## 15. Common Problems and Solutions

### Problem 1: Table does not exist

Check whether migrations have run:

```bash
php artisan migrate:status
```

If necessary, run:

```bash
php artisan migrate
```

### Problem 2: Mass assignment exception

Check the `$fillable` property in:

```text
app/Models/Task.php
```

Make sure the attributes being passed to `Task::create()` and `update()` are allowed.

### Problem 3: Route not found

Check registered routes:

```bash
php artisan route:list --path=tasks
```

Clear the route cache if necessary:

```bash
php artisan route:clear
```

### Problem 4: Page returns 404

Check whether the requested task ID exists in the database.

Laravel's route model binding returns 404 when the task cannot be found.

### Problem 5: CSRF token mismatch

Make sure every POST, PUT, PATCH, and DELETE form includes:

```php
@csrf
```

For PUT and DELETE forms, also include the appropriate `@method()` directive.

---

## 16. Knowledge Check

1. What does CRUD stand for?
2. What is a resource controller?
3. Which Artisan command creates a resource controller?
4. What does `Route::resource()` do?
5. What is the purpose of the `store()` method?
6. What is route model binding?
7. Why do we use `@csrf`?
8. Why do we use `@method('PUT')`?
9. What is the purpose of `$task->update()`?
10. What is the difference between `destroy()` and `delete()` in this example?

---

## 17. Completion Checklist

- [ ] Created a resource controller.
- [ ] Registered resource routes.
- [ ] Understood route model binding.
- [ ] Created a reusable task form.
- [ ] Implemented Create functionality.
- [ ] Implemented Read functionality.
- [ ] Implemented Update functionality.
- [ ] Implemented Delete functionality.
- [ ] Added basic form validation.
- [ ] Displayed success messages.
- [ ] Tested all CRUD operations using MySQL.

---

## 18. Official Laravel Documentation

- [Controllers](https://laravel.com/docs/13.x/controllers)
- [Resource Controllers](https://laravel.com/docs/13.x/controllers#resource-controllers)
- [Routing](https://laravel.com/docs/13.x/routing)
- [Eloquent ORM](https://laravel.com/docs/13.x/eloquent)
- [Validation](https://laravel.com/docs/13.x/validation)
- [Blade Templates](https://laravel.com/docs/13.x/blade)

---

## Next Lesson: Lesson 09 – Form Validation and Error Handling

In the next lesson, we will learn:

- Laravel validation in detail.
- Required, email, string, numeric, and date rules.
- Custom validation messages.
- Displaying validation errors in Blade.
- Preserving submitted form values.
- Handling validation failures.
- Improving the Task Management System forms.