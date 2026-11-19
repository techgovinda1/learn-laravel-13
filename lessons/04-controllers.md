# Lesson 04: Controllers and Request Handling

**Course:** Learn Laravel 13 – Practical Beginner Course  
**Level:** Beginner  
**Duration:** 40–50 minutes  
**Project:** Task Management System

---

## 1. Learning Objectives

By the end of this lesson, you will be able to:

- Understand the role of controllers in Laravel.
- Create controllers using Artisan.
- Connect routes to controller methods.
- Pass data from controllers to Blade views.
- Understand Laravel's Request object.
- Handle basic form submissions.

---

## 2. What is a Controller?

A controller handles application requests and organizes the logic behind them.

Instead of writing all application logic inside `routes/web.php`, we move it into controller methods.

For example:

**Without a controller:**

```php
Route::get('/tasks', function () {
    return 'Task List';
});
```

**With a controller:**

```php
Route::get('/tasks', [TaskController::class, 'index']);
```

The second approach keeps routes clean and makes application logic easier to maintain.

### MVC reminder

Laravel follows the MVC architectural pattern:

| Component | Responsibility |
|---|---|
| Model | Handles data and database operations |
| View | Displays the user interface |
| Controller | Handles requests and application logic |

---

## 3. Create Your First Controller

Open your project directory in the terminal:

```bash
cd C:\wamp64\www\laravel\task-manager
```

Run:

```bash
php artisan make:controller TaskController
```

Laravel creates:

```text
app/
└── Http/
    └── Controllers/
        └── TaskController.php
```

Open `TaskController.php`.

Replace its contents with:

```php
<?php

namespace App\Http\Controllers;

class TaskController extends Controller
{
    public function index()
    {
        return 'Welcome to the Task Manager';
    }

    public function show(string $id)
    {
        return "Viewing task ID: {$id}";
    }
}
```

We have created two controller methods:

- `index()` displays the task list.
- `show()` displays an individual task.

---

## 4. Connect Routes to the Controller

Open:

```text
routes/web.php
```

Add:

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\TaskController;

Route::get('/tasks', [TaskController::class, 'index'])
    ->name('tasks.index');

Route::get('/tasks/{id}', [TaskController::class, 'show'])
    ->whereNumber('id')
    ->name('tasks.show');
```

Visit these URLs:

| URL | Expected Output |
|---|---|
| `/tasks` | Welcome to the Task Manager |
| `/tasks/1` | Viewing task ID: 1 |
| `/tasks/5` | Viewing task ID: 5 |

The `whereNumber('id')` constraint ensures that the task ID must be numeric.

---

## 5. Pass Data from Controller to View

Returning plain text is useful for testing, but real applications usually display Blade views.

Let's create a task list page.

### Step 1: Update the controller

Open:

```text
app/Http/Controllers/TaskController.php
```

Update the `index()` method:

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

`compact('tasks')` passes the `$tasks` variable to the view.

### Step 2: Create the Blade view

Create this directory:

```text
resources/views/tasks
```

Inside it, create:

```text
index.blade.php
```

Add:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Task Manager</title>
</head>
<body>

    <h1>My Tasks</h1>

    @foreach ($tasks as $task)
        <div>
            <h3>{{ $task['title'] }}</h3>
            <p>Status: {{ $task['status'] }}</p>
            <a href="{{ route('tasks.show', $task['id']) }}">
                View Task
            </a>
        </div>
        <hr>
    @endforeach

</body>
</html>
```

### Step 3: Update the show method

For now, keep the individual task page simple:

```php
public function show(string $id)
{
    return "Viewing task ID: {$id}";
}
```

Visit:

```text
http://localhost:8000/tasks
```

You should see three tasks with their titles, statuses, and View Task links.

**Important:** These tasks are currently stored in a PHP array. We will connect tasks to a database in a later lesson.

---

## 6. Understanding Laravel's Request Object

The Request object provides access to information sent by the browser.

For example, it can retrieve:

- Form input values.
- Query string parameters.
- Request data.
- Uploaded files.

Import the Request class:

```php
use Illuminate\Http\Request;
```

### Example: Read a query parameter

Add this method to `TaskController`:

```php
public function search(Request $request)
{
    $keyword = $request->query('keyword', '');

    return "Searching for: {$keyword}";
}
```

Add the route to `web.php`:

```php
Route::get('/search', [TaskController::class, 'search'])
    ->name('tasks.search');
```

Visit:

```text
http://localhost:8000/search?keyword=Laravel
```

Expected output:

```text
Searching for: Laravel
```

The `query()` method retrieves a value from the URL query string.

---

## 7. Handle a Form Submission

Now let's create a simple form to submit a new task title.

We will display the submitted value without saving it to the database.

### Step 1: Add a form view

Create:

```text
resources/views/tasks/create.blade.php
```

Add:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Create Task</title>
</head>
<body>

    <h1>Create a New Task</h1>

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

</body>
</html>
```

### Step 2: Add controller methods

Add these methods to `TaskController`:

```php
public function create()
{
    return view('tasks.create');
}

public function store(Request $request)
{
    $title = $request->input('title');

    return "Task submitted: {$title}";
}
```

### Step 3: Register the routes

Add these routes to `routes/web.php`:

```php
Route::get('/tasks/create', [TaskController::class, 'create'])
    ->name('tasks.create');

Route::post('/tasks', [TaskController::class, 'store'])
    ->name('tasks.store');
```

### Step 4: Test the form

Visit:

```text
http://localhost:8000/tasks/create
```

Enter a task title and submit the form.

Expected output:

```text
Task submitted: Learn Laravel
```

### Understanding the code

| Code | Purpose |
|---|---|
| `Request $request` | Receives the incoming request |
| `$request->input('title')` | Retrieves the submitted title |
| `method="POST"` | Sends form data using POST |
| `@csrf` | Adds Laravel's CSRF protection token |
| `route('tasks.store')` | Generates the URL for the named route |

**Note:** This example does not save tasks permanently. Database storage and validation will be covered in later lessons.

---

## 8. Inspect Your Routes

Run:

```bash
php artisan route:list
```

To show only task-related routes:

```bash
php artisan route:list --path=tasks
```

You should find routes for:

- Listing tasks.
- Viewing an individual task.
- Displaying the create form.
- Submitting a new task.

---

## 9. Practical Assignment

Extend the Task Management System.

### Tasks

1. Create a `TaskController` if you have not already created one.
2. Create an `index()` method to display at least five sample tasks.
3. Create a `show()` method to display a task ID.
4. Create a `create()` method to display the task form.
5. Create a `store()` method to receive the submitted title.
6. Add a search method that reads a keyword from the URL.
7. Register all routes using controller methods.

### Expected URLs

| URL | Expected Result |
|---|---|
| `/tasks` | Display all sample tasks |
| `/tasks/1` | Display task ID 1 |
| `/tasks/create` | Display the create form |
| `/search?keyword=PHP` | Display the search keyword |

---

## 10. Knowledge Check

1. What is the purpose of a controller?
2. Which Artisan command creates a controller?
3. Where are controllers stored?
4. How do you connect a route to a controller method?
5. What does `return view()` do?
6. How do you pass data from a controller to a view?
7. What is the purpose of the Request object?
8. What does `$request->input('title')` retrieve?
9. Why do we use `@csrf` in forms?
10. What is the difference between `query()` and `input()`?

---

## 11. Completion Checklist

- [ ] Created `TaskController`.
- [ ] Connected routes to controller methods.
- [ ] Created a Blade view for tasks.
- [ ] Passed sample data from controller to view.
- [ ] Used route parameters.
- [ ] Retrieved a query parameter.
- [ ] Created a task submission form.
- [ ] Handled a POST request.
- [ ] Checked routes using Artisan.

---

## 12. Official Laravel Documentation

- [Controllers](https://laravel.com/docs/13.x/controllers)
- [Requests](https://laravel.com/docs/13.x/requests)
- [Routing](https://laravel.com/docs/13.x/routing)
- [Blade Templates](https://laravel.com/docs/13.x/blade)

---

## Next Lesson: Lesson 05 – Blade Templates and Layouts

In the next lesson, we will learn:

- Blade template syntax.
- Displaying variables and data.
- Conditional statements.
- Loops and reusable components.
- Template inheritance using layouts.
- Building a consistent layout for the Task Management System.