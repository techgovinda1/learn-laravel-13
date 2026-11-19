# Lesson 02: Understanding Laravel Project Structure

**Course:** Learn Laravel 13 – Practical Beginner Course
**Level:** Beginner
**Duration:** 30–40 minutes
**Project:** Task Management System

---

## 1. Learning Objectives

By the end of this lesson, you will be able to:

- Understand Laravel's directory structure.
- Identify important folders and files.
- Understand the MVC architecture.
- Locate routes, controllers, models and views.
- Understand where to write code when building Laravel applications.

## 2. Explore the Laravel Project

Open your `task-manager` project in VS Code.

```bash
code .
```

A Laravel project contains several directories and files.

```text
task-manager/
│
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   └── Middleware/
│   ├── Models/
│   └── Providers/
│
├── bootstrap/
├── config/
│
├── database/
│   ├── factories/
│   ├── migrations/
│   └── seeders/
│
├── public/
│
├── resources/
│   ├── css/
│   ├── js/
│   └── views/
│
├── routes/
│   ├── web.php
│   └── console.php
│
├── storage/
├── tests/
├── vendor/
│
├── .env
├── artisan
├── composer.json
└── composer.lock
```

Let's understand the important directories.

## 3. Important Laravel Directories

### 3.1 `app/` – Application Logic

This directory contains most of your application's PHP code.

Important subdirectories:

| Directory               | Purpose                        |
| ----------------------- | ------------------------------ |
| `app/Http/Controllers/` | Handles HTTP requests          |
| `app/Http/Middleware/`  | Filters HTTP requests          |
| `app/Models/`           | Represents database entities   |
| `app/Providers/`        | Registers application services |

Example:

When building our Task Management System, we will create:

```text
app/
├── Http/
│   └── Controllers/
│       └── TaskController.php
│
└── Models/
    └── Task.php
```

### 3.2 `routes/` – Application URLs

This directory defines the URLs users can access.

Important files:

- `web.php` – Web application routes.
- `console.php` – Console commands.

API routes are available through `routes/api.php` when API routing is installed.

**Practical: Create your first route**

Open `routes/web.php` and add:

```php
use Illuminate\Support\Facades\Route;

Route::get('/hello', function () {
    return 'Hello Laravel!';
});
```

Start the development server:

```bash
php artisan serve
```

Visit:

[http://127.0.0.1:8000/hello](http://127.0.0.1:8000/hello)

**Expected output:**

```text
Hello Laravel!
```

**What happened?**

1. The browser requested `/hello`.
2. Laravel checked `routes/web.php`.
3. Laravel found the matching GET route.
4. The closure executed.
5. Laravel returned the text response.

### 3.3 `resources/views/` – Frontend Templates

This directory contains Blade templates.

Blade is Laravel's templating engine for creating dynamic HTML pages.

**Practical: Create your first Blade view**

Create a file:

`resources/views/welcome-message.blade.php`

Add:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Laravel Tutorial</title>
</head>
<body>

    <h1>Welcome to Laravel</h1>

    <p>We are learning Laravel practically.</p>

</body>
</html>
```

Now update `routes/web.php`:

```php
Route::get('/welcome', function () {
    return view('welcome-message');
});
```

Visit:

[http://127.0.0.1:8000/welcome](http://127.0.0.1:8000/welcome)

You should see your custom HTML page.

**Remember:** The `view()` function loads a Blade template from `resources/views`.

### 3.4 `database/` – Database Files

This directory contains three important folders:

| Folder        | Purpose                         |
| ------------- | ------------------------------- |
| `migrations/` | Defines database tables         |
| `seeders/`    | Inserts sample database records |
| `factories/`  | Generates sample model data     |

We will use these when creating our Task Management System database.

### 3.5 `config/` – Application Configuration

Contains configuration files for:

- Database connections.
- Application settings.
- Cache.
- Mail.
- Authentication.
- Filesystems.

Most environment-specific settings should be configured through `.env`.

### 3.6 `public/` – Public Entry Point

This directory contains the public entry point of your application.

Important file:

```text
public/index.php
```

When a web request enters Laravel, it eventually passes through this file, which bootstraps the application.

Static assets such as images, CSS and JavaScript can also be served from this directory.

### 3.7 `storage/` – Application Storage

Used for storing:

- Application logs.
- Cached files.
- Compiled Blade templates.
- Uploaded or generated files.

For example, Laravel application logs are generally stored in:

```text
storage/logs/laravel.log
```

### 3.8 `vendor/` – Dependencies

This directory contains third-party PHP packages installed through Composer.

For example, Laravel and its supporting libraries are installed here.

**Important:** Do not manually edit files inside `vendor/`.

### 3.9 `.env` – Environment Configuration

The `.env` file stores environment-specific application settings.

Example:

```env
APP_NAME=TaskManager
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=task_manager
DB_USERNAME=root
DB_PASSWORD=
```

This allows you to change database credentials and application settings without changing your main application code.

**Security reminder:** Never commit your `.env` file to Git.

## 4. Understand MVC Architecture

Laravel follows the Model-View-Controller (MVC) architectural pattern.

| Component  | Responsibility              | Laravel Location        |
| ---------- | --------------------------- | ----------------------- |
| Model      | Handles database data       | `app/Models/`           |
| View       | Displays the user interface | `resources/views/`      |
| Controller | Handles application logic   | `app/Http/Controllers/` |

### Practical Example

Imagine a user visits:

`http://127.0.0.1:8000/tasks`

The request follows this flow:

```text
Browser
   |
   v
Route (web.php)
   |
   v
TaskController
   |
   v
Task Model
   |
   v
Database
   |
   v
TaskController
   |
   v
Blade View
   |
   v
Browser
```

We will implement this complete process in upcoming lessons.

## 5. Practical Exercises

### Exercise 1: Create an About Route

Open `routes/web.php` and add:

```php
Route::get('/about', function () {
    return 'This is my first Laravel application.';
});
```

Visit:

[http://127.0.0.1:8000/about](http://127.0.0.1:8000/about)

**Expected result:** The browser displays the text.

### Exercise 2: Create an About View

Create:

`resources/views/about.blade.php`

Add:

```html
<h1>About Our Application</h1>

<p>We are building a Task Management System using Laravel.</p>
```

### Exercise 3: Connect the Route to the View

Update the `/about` route:

```php
Route::get('/about', function () {
    return view('about');
});
```

Refresh the browser.

You should now see the HTML page instead of the plain text response.

### Exercise 4: Explore Laravel Directories

Find the following locations in your project:

- [ ] `app/Models`
- [ ] `app/Http/Controllers`
- [ ] `routes/web.php`
- [ ] `resources/views`
- [ ] `database/migrations`
- [ ] `storage/logs`
- [ ] `.env`

## 6. Knowledge Check

Answer these questions before moving to the next lesson.

1. Where do we create controllers in Laravel?
2. Where are Blade templates stored?
3. What is the purpose of `routes/web.php`?
4. What is the purpose of the `app/Models` directory?
5. Where are database migrations stored?
6. What is the purpose of the `.env` file?
7. What is MVC architecture?
8. Why should we avoid editing files inside `vendor/`?

## 7. Lesson Completion Checklist

- [ ] Explored the Laravel project structure.
- [ ] Identified important Laravel directories.
- [ ] Created a custom route.
- [ ] Created a Blade view.
- [ ] Connected a route to a view.
- [ ] Understood MVC architecture.
- [ ] Completed the practical exercises.

## 8. Official Documentation

Read the following documentation for additional understanding:

- [Laravel 13 – Directory Structure](https://laravel.com/docs/13.x/structure)
- [Laravel 13 – Routing](https://laravel.com/docs/13.x/routing)
- [Laravel 13 – Views](https://laravel.com/docs/13.x/views)
- [Laravel 13 – Configuration](https://laravel.com/docs/13.x/configuration)

---

## Next Lesson

**Lesson 03: Routing and HTTP Requests**

In the next lesson, we will learn:

- GET, POST, PUT and DELETE requests.
- Route parameters.
- Named routes.
- Route groups.
- Practical route implementation.

We will continue building our Task Management System step by step.
