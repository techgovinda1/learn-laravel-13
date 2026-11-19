# Lesson 03: Routing and HTTP Requests

**Course:** Learn Laravel 13 – Practical Beginner Course  
**Level:** Beginner  
**Duration:** 40–45 minutes  
**Project:** Task Management System

---

## 1. Learning Objectives

By the end of this lesson, you will be able to:

- Understand how routing works in Laravel.
- Understand HTTP methods: GET, POST, PUT, PATCH and DELETE.
- Create routes and route parameters.
- Use named routes.
- Understand route groups.
- Pass data through URLs.
- Inspect registered routes using Artisan.

## 2. What is Routing?

Routing determines how Laravel responds when a user visits a particular URL.

For example:

| URL | Purpose |
|---|---|
| `/` | Home page |
| `/about` | About page |
| `/tasks` | Display all tasks |
| `/tasks/create` | Display task creation form |
| `/contact` | Contact page |

Laravel routes are generally defined in:

```text
routes/web.php
```

### Basic Route Syntax

```php
Route::get('/hello', function () {
    return 'Hello Laravel!';
});
```

Explanation:

- `Route::get()` defines a GET route.
- `/hello` is the URL.
- The function executes when the URL is requested.
- `return` sends a response to the browser.

**Practical:**

Add this route to `routes/web.php`:

```php
Route::get('/hello', function () {
    return 'Hello from Laravel!';
});
```

Visit:

http://127.0.0.1:8000/hello

Expected output:

```text
Hello from Laravel!
```

## 3. Understanding HTTP Methods

HTTP methods define what action a request should perform.

Laravel supports several HTTP methods.

| Method | Purpose | Example |
|---|---|---|
| GET | Retrieve information | Display tasks |
| POST | Submit or create data | Create a task |
| PUT | Replace/update a resource | Update a task |
| PATCH | Partially update a resource | Change task status |
| DELETE | Delete a resource | Delete a task |

### 3.1 GET Route

Used to retrieve information.

```php
Route::get('/tasks', function () {
    return 'Display all tasks';
});
```

Visit:

`http://127.0.0.1:8000/tasks`

### 3.2 POST Route

Used to submit data to the server.

```php
Route::post('/tasks', function () {
    return 'Task submitted successfully';
});
```

POST requests are generally submitted through HTML forms or API clients, rather than directly by entering a URL in the browser.

### 3.3 PUT Route

Used to update an existing resource.

```php
Route::put('/tasks/{id}', function ($id) {
    return 'Updating task ID: ' . $id;
});
```

### 3.4 PATCH Route

Used to partially update an existing resource.

```php
Route::patch('/tasks/{id}', function ($id) {
    return 'Updating task status: ' . $id;
});
```

### 3.5 DELETE Route

Used to delete a resource.

```php
Route::delete('/tasks/{id}', function ($id) {
    return 'Deleting task ID: ' . $id;
});
```

**Important:** You cannot normally test POST, PUT, PATCH and DELETE routes by typing their URLs into the browser. We will use HTML forms and route testing in later lessons.

## 4. Route Parameters

Route parameters allow us to pass dynamic values through URLs.

For example:

```text
/tasks/1
/tasks/2
/tasks/3
```

Here, the task ID changes depending on the requested task.

### 4.1 Required Parameters

Example:

```php
Route::get('/tasks/{id}', function ($id) {
    return 'Task ID: ' . $id;
});
```

Visit:

`http://127.0.0.1:8000/tasks/5`

Expected output:

```text
Task ID: 5
```

Another example:

```php
Route::get('/users/{id}/posts/{postId}', function ($id, $postId) {
    return "User ID: $id, Post ID: $postId";
});
```

Visit:

`http://127.0.0.1:8000/users/10/posts/25`

Expected output:

```text
User ID: 10, Post ID: 25
```

### 4.2 Optional Parameters

Sometimes a route parameter may not be provided.

We can make it optional using `?`.

```php
Route::get('/welcome/{name?}', function ($name = 'Guest') {
    return "Welcome, $name!";
});
```

Try:

```text
http://127.0.0.1:8000/welcome
```

Output:

```text
Welcome, Guest!
```

Now visit:

```text
http://127.0.0.1:8000/welcome/John
```

Output:

```text
Welcome, John!
```

### 4.3 Route Constraints

We can restrict route parameters using regular expressions.

Example:

```php
Route::get('/tasks/{id}', function ($id) {
    return "Task ID: $id";
})->whereNumber('id');
```

Now `/tasks/5` works, but `/tasks/abc` does not match this route.

Other useful constraints:

```php
->whereAlpha('name');

->whereAlphaNumeric('name');

->whereNumber('id');
```

## 5. Named Routes

Named routes allow us to reference routes using a name instead of writing their URLs repeatedly.

Example:

```php
Route::get('/tasks', function () {
    return 'Task List';
})->name('tasks.index');
```

Instead of hardcoding `/tasks`, we can generate its URL using:

```php
route('tasks.index');
```

### Practical Example

Add these routes:

```php
Route::get('/tasks', function () {
    return 'All Tasks';
})->name('tasks.index');

Route::get('/tasks/create', function () {
    return 'Create Task Form';
})->name('tasks.create');
```

Now create a simple Blade file:

`resources/views/task-links.blade.php`

```html
<h1>Task Management</h1>

<a href="{{ route('tasks.index') }}">
    View Tasks
</a>

<br><br>

<a href="{{ route('tasks.create') }}">
    Create Task
</a>
```

Add another route:

```php
Route::get('/task-links', function () {
    return view('task-links');
});
```

Visit:

`http://127.0.0.1:8000/task-links`

Click both links.

**Why use named routes?**

If the URL changes later, you do not need to update every link manually. Laravel generates the correct URL using the route name.

## 6. Route Groups

Route groups allow us to organize related routes and apply shared settings.

For example, imagine we have multiple task-related routes.

Instead of writing individual prefixes repeatedly, we can use a route group.

```php
Route::prefix('admin')->group(function () {

    Route::get('/dashboard', function () {
        return 'Admin Dashboard';
    });

    Route::get('/users', function () {
        return 'Manage Users';
    });

});
```

These routes become:

```text
/admin/dashboard
/admin/users
```

### Named Route Groups

We can also assign a common name prefix:

```php
Route::name('admin.')->group(function () {

    Route::get('/dashboard', function () {
        return 'Admin Dashboard';
    })->name('dashboard');

    Route::get('/users', function () {
        return 'Manage Users';
    })->name('users');

});
```

Their route names become:

```text
admin.dashboard
admin.users
```

### Middleware Route Groups

We can apply middleware to multiple routes.

```php
Route::middleware('auth')->group(function () {

    Route::get('/dashboard', function () {
        return 'Dashboard';
    });

    Route::get('/profile', function () {
        return 'User Profile';
    });

});
```

These routes require an authenticated user when the authentication middleware is configured.

We will explore middleware and authentication in a later lesson.

## 7. Resource Routes

Laravel provides a convenient way to define all standard CRUD routes using a single command.

Example:

```php
use App\Http\Controllers\TaskController;

Route::resource('tasks', TaskController::class);
```

This generates seven routes automatically.

| HTTP Method | URL | Controller Method | Route Name |
|---|---|---|---|
| GET | `/tasks` | index | tasks.index |
| GET | `/tasks/create` | create | tasks.create |
| POST | `/tasks` | store | tasks.store |
| GET | `/tasks/{task}` | show | tasks.show |
| GET | `/tasks/{task}/edit` | edit | tasks.edit |
| PUT/PATCH | `/tasks/{task}` | update | tasks.update |
| DELETE | `/tasks/{task}` | destroy | tasks.destroy |

**Important:** We will implement these routes with a real controller when we build CRUD functionality.

## 8. Inspect Routes Using Artisan

Laravel provides a command to display all registered routes.

Run:

```bash
php artisan route:list
```

To filter routes:

```bash
php artisan route:list --path=tasks
```

To display additional middleware information:

```bash
php artisan route:list -v
```

This is especially useful when debugging routing problems.

## 9. Practical Assignment

Let's create a small routing system for our Task Management application.

Open:

`routes/web.php`

Add the following routes:

```php
use Illuminate\Support\Facades\Route;

// Home
Route::get('/', function () {
    return 'Task Management System';
})->name('home');

// Task listing
Route::get('/tasks', function () {
    return 'All Tasks';
})->name('tasks.index');

// Create task
Route::get('/tasks/create', function () {
    return 'Create New Task';
})->name('tasks.create');

// Show individual task
Route::get('/tasks/{id}', function ($id) {
    return "Viewing Task ID: $id";
})->whereNumber('id')->name('tasks.show');

// About page
Route::get('/about', function () {
    return 'About Our Application';
})->name('about');

// Contact page
Route::get('/contact', function () {
    return 'Contact Us';
})->name('contact');
```

**Test the following URLs:**

| URL | Expected Result |
|---|---|
| `/` | Task Management System |
| `/tasks` | All Tasks |
| `/tasks/create` | Create New Task |
| `/tasks/1` | Viewing Task ID: 1 |
| `/about` | About Our Application |
| `/contact` | Contact Us |

Also run:

```bash
php artisan route:list
```

Check whether all your routes are registered.

## 10. Knowledge Check

Answer these questions:

1. What is routing in Laravel?
2. What is the difference between GET and POST?
3. What is the purpose of PUT and PATCH?
4. How do you define a route parameter?
5. How do you define an optional route parameter?
6. What are named routes?
7. What is the purpose of route groups?
8. What does `Route::resource()` do?
9. Which Artisan command displays all registered routes?
10. Why should we use route names instead of hardcoding URLs?

## 11. Lesson Completion Checklist

- [ ] Created GET routes.
- [ ] Understood POST, PUT, PATCH and DELETE.
- [ ] Created required route parameters.
- [ ] Created optional route parameters.
- [ ] Applied route constraints.
- [ ] Created named routes.
- [ ] Created route groups.
- [ ] Explored resource routes.
- [ ] Used `php artisan route:list`.
- [ ] Completed the practical assignment.

## 12. Official Documentation

Read the following official Laravel documentation:

- [Laravel 13 – Routing](https://laravel.com/docs/13.x/routing)
- [Laravel 13 – Controllers](https://laravel.com/docs/13.x/controllers)
- [Laravel 13 – Requests](https://laravel.com/docs/13.x/requests)

---

## Next Lesson

**Lesson 04: Controllers and Request Handling**

In the next lesson, we will:

- Understand controllers and their responsibilities.
- Create controllers using Artisan.
- Move route logic into controllers.
- Pass data from controllers to views.
- Understand the Request object.
- Build our first controller-based application feature.

We will continue developing the Task Management System step by step.
