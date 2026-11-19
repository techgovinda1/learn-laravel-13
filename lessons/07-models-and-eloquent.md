# Lesson 07: Models and Eloquent ORM

**Course:** Learn Laravel 13 – Practical Beginner Course  
**Level:** Beginner  
**Duration:** 45–60 minutes  
**Project:** Task Management System

---

## 1. Learning Objectives

By the end of this lesson, you will be able to:

- Understand Laravel models and Eloquent ORM.
- Create models using Artisan.
- Connect models to database tables.
- Retrieve records from a database.
- Insert new records.
- Update existing records.
- Delete records.
- Understand basic Eloquent query methods.

---

## 2. What is a Model?

A model represents a database table in your Laravel application.

For example:

| Model | Database Table |
|---|---|
| Task | tasks |
| User | users |
| Product | products |
| Category | categories |

Laravel uses a naming convention:

- Model names are singular and use PascalCase.
- Database table names are plural and use snake_case.

For example:

```text
Task.php  →  tasks
User.php  →  users
Product.php  →  products
```

Models are normally stored inside:

```text
app/Models/
```

---

## 3. What is Eloquent ORM?

Eloquent is Laravel's Object-Relational Mapper (ORM).

It allows you to interact with database records using PHP objects instead of writing SQL queries for every operation.

**Traditional SQL:**

```sql
SELECT * FROM tasks;
```

**Using Eloquent:**

```php
Task::all();
```

Eloquent makes database operations easier to read and maintain.

---

## 4. Create a Task Model

Open your Laravel project terminal.

Run:

```bash
php artisan make:model Task
```

Laravel creates:

```text
app/
└── Models/
    └── Task.php
```

Open `app/Models/Task.php`.

You should see a model similar to:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Task extends Model
{
    //
}
```

Laravel automatically associates the `Task` model with the `tasks` database table.

You do not need to specify the table name because Laravel follows its naming conventions.

---

## 5. Configure Mass Assignment

Before inserting records using an array, configure which attributes can be mass assigned.

Update `app/Models/Task.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Task extends Model
{
    protected $fillable = [
        'title',
        'description',
        'status',
        'priority',
        'due_date',
        'category',
    ];
}
```

### What is `$fillable`?

`$fillable` defines which model attributes can be assigned through mass-assignment operations such as:

```php
Task::create($data);
```

This helps protect your application from unintended mass assignment.

**Important:** Only include attributes that users or application features should be allowed to assign. Do not blindly make every database column fillable.

---

## 6. Retrieve Records from the Database

Let's retrieve tasks using Eloquent.

Open:

```text
app/Http/Controllers/TaskController.php
```

Import the Task model:

```php
use App\Models\Task;
```

### Retrieve all tasks

```php
public function index()
{
    $tasks = Task::all();

    return view('tasks.index', compact('tasks'));
}
```

`Task::all()` retrieves all records from the tasks table.

### Retrieve the first record

```php
$task = Task::first();
```

### Find a task by ID

```php
$task = Task::find(1);
```

### Find a task or return a 404 response

```php
$task = Task::findOrFail(1);
```

If the requested record does not exist, `findOrFail()` automatically throws a model-not-found exception, which Laravel handles as a 404 response.

### Retrieve tasks with a condition

```php
$tasks = Task::where('status', 'Pending')->get();
```

### Retrieve tasks in descending order

```php
$tasks = Task::orderBy('created_at', 'desc')->get();
```

### Limit the results

```php
$tasks = Task::limit(5)->get();
```

---

## 7. Create Records Using Eloquent

There are different ways to create records.

### Method 1: Using create()

```php
Task::create([
    'title' => 'Learn Eloquent',
    'description' => 'Understand Laravel models',
    'status' => 'Pending',
    'priority' => 'High',
]);
```

This uses mass assignment, so the attributes must be allowed by `$fillable`.

### Method 2: Using a model instance

```php
$task = new Task();

$task->title = 'Learn Laravel Models';
$task->description = 'Practice Eloquent';
$task->status = 'Pending';
$task->priority = 'Normal';

$task->save();
```

This method assigns attributes individually and then saves the model.

---

## 8. Update Records

Find a task and update its information:

```php
$task = Task::find(1);

if ($task) {
    $task->title = 'Learn Laravel Eloquent';
    $task->status = 'Completed';

    $task->save();
}
```

You can also use `update()`:

```php
$task = Task::findOrFail(1);

$task->update([
    'status' => 'Completed',
    'priority' => 'High',
]);
```

The `update()` method uses mass assignment and respects `$fillable`.

---

## 9. Delete Records

Find and delete a task:

```php
$task = Task::find(1);

if ($task) {
    $task->delete();
}
```

Or use:

```php
Task::destroy(1);
```

`destroy()` deletes the record with the specified primary key.

**Important:** Deleting a record is a permanent database operation unless you have configured soft deletes.

---

## 10. Practical: Connect the Task List to MySQL

In Lesson 05, we displayed tasks using a PHP array.

Now we will retrieve them from the database.

Open:

```text
app/Http/Controllers/TaskController.php
```

Update the `index()` method:

```php
use App\Models\Task;

public function index()
{
    $tasks = Task::all();

    return view('tasks.index', compact('tasks'));
}
```

Make sure the model import is placed at the top of the file with the other imports.

Update:

```text
resources/views/tasks/index.blade.php
```

Use:

```php
@extends('layouts.app')

@section('title', 'My Tasks')

@section('content')

    <h2>My Tasks</h2>

    @forelse ($tasks as $task)

        <div>
            <h3>{{ $task->title }}</h3>

            <p>Status: {{ $task->status }}</p>

            <p>Priority: {{ $task->priority }}</p>

            <a href="{{ route('tasks.show', $task->id) }}">
                View Task
            </a>
        </div>

        <hr>

    @empty

        <p>No tasks available.</p>

    @endforelse

@endsection
```

Notice the difference:

**Array data:**

```php
$task['title']
```

**Eloquent model data:**

```php
$task->title
```

Eloquent returns model objects, so we access attributes using object notation.

---

## 11. Insert Your First Database Record

You can insert a record using Laravel Tinker.

Tinker is an interactive command-line environment for working with your Laravel application.

Run:

```bash
php artisan tinker
```

Inside Tinker, execute:

```php
use App\Models\Task;
```

Create a task:

```php
Task::create([
    'title' => 'Learn Eloquent ORM',
    'description' => 'Practice database operations',
    'status' => 'Pending',
    'priority' => 'High',
]);
```

Retrieve all tasks:

```php
Task::all();
```

Find a task:

```php
Task::find(1);
```

Exit Tinker:

```php
exit
```

Now visit:

```text
http://localhost:8000/tasks
```

Your database record should appear on the page.

**Note:** If you already have records, the new task may not have ID 1. Check the returned record or database table for its actual ID.

---

## 12. Common Eloquent Query Methods

| Method | Purpose |
|---|---|
| `all()` | Retrieve all records |
| `find($id)` | Find a record by primary key |
| `findOrFail($id)` | Find a record or return 404 |
| `first()` | Retrieve the first matching record |
| `where()` | Filter records |
| `orderBy()` | Sort records |
| `latest()` | Sort by latest creation date by default |
| `limit()` | Limit the number of results |
| `get()` | Execute a query and retrieve results |
| `create()` | Create a new record |
| `save()` | Save a model instance |
| `update()` | Update model attributes |
| `delete()` | Delete a model record |
| `destroy()` | Delete records by primary key |

### Example: Retrieve pending tasks

```php
$tasks = Task::where('status', 'Pending')->get();
```

### Example: Retrieve high-priority tasks

```php
$tasks = Task::where('priority', 'High')->get();
```

### Example: Retrieve the latest five tasks

```php
$tasks = Task::latest()->limit(5)->get();
```

---

## 13. Practical Assignment

Continue developing the Task Management System.

### Tasks

1. Create the `Task` model.
2. Configure `$fillable`.
3. Retrieve all tasks from MySQL.
4. Display database records in the task list.
5. Insert at least three tasks using Tinker.
6. Retrieve a task using its ID.
7. Retrieve tasks with `Pending` status.
8. Retrieve tasks with `High` priority.
9. Update a task's status.
10. Delete a test task.

### Expected Result

Your task list should display actual records from the MySQL database rather than a hardcoded PHP array.

---

## 14. Knowledge Check

1. What is a model in Laravel?
2. What is Eloquent ORM?
3. Where are Laravel models stored?
4. How does Laravel associate a model with a database table?
5. What is the purpose of `$fillable`?
6. What is the difference between `find()` and `findOrFail()`?
7. What does `Task::all()` return?
8. What is the difference between `create()` and `save()`?
9. How do you update an existing record?
10. What is Laravel Tinker used for?

---

## 15. Completion Checklist

- [ ] Created the Task model.
- [ ] Understood Eloquent ORM.
- [ ] Configured `$fillable`.
- [ ] Retrieved database records.
- [ ] Inserted records using Tinker.
- [ ] Updated a database record.
- [ ] Deleted a test record.
- [ ] Connected the task list to MySQL.
- [ ] Displayed database records using Blade.

---

## 16. Official Laravel Documentation

- [Eloquent ORM](https://laravel.com/docs/13.x/eloquent)
- [Database: Getting Started](https://laravel.com/docs/13.x/database)
- [Laravel Tinker](https://laravel.com/docs/13.x/artisan#tinker)

---

## Next Lesson: Lesson 08 – CRUD Operations in Laravel

In the next lesson, we will build complete task management functionality:

- Create tasks using forms.
- Display tasks from the database.
- View individual task details.
- Edit existing tasks.
- Delete tasks.
- Use resource controllers and resource routes.
- Connect forms, controllers, models, and database operations.

We will turn the current Task Management System into a functional CRUD application.