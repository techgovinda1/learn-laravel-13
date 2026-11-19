# Lesson 01: Introduction to Laravel & Project Setup

**Course:** Learn Laravel 13 – Practical Beginner Course  
**Level:** Beginner  
**Duration:** 30–45 minutes  
**Project:** Task Management System

---

## 1. Learning Objectives

By the end of this lesson, you will be able to:

- Understand what Laravel is.
- Understand why developers use Laravel.
- Identify the basic requirements for Laravel development.
- Create a new Laravel 13 project.
- Run a Laravel application locally.
- Initialize a Git repository and commit your project.

## 2. What is Laravel?

Laravel is an open-source PHP web application framework designed to make web development easier, faster and more structured.

It provides built-in features for:

- Routing and request handling.
- Database operations using Eloquent ORM.
- Authentication and authorization.
- Form validation.
- Middleware.
- Blade templating.
- API development.
- Caching and background jobs.

Instead of writing everything from scratch, Laravel provides reusable tools and conventions.

**Example:** If you want to build an employee management system, Laravel provides tools to manage employees, validate forms, connect to databases and handle user authentication.

### Why use Laravel?

| Without a framework | With Laravel |
|---|---|
| Manually manage URLs | Built-in routing |
| Write database queries manually | Eloquent ORM |
| Build form validation manually | Built-in validation |
| Create reusable templates manually | Blade templating |
| Implement security features from scratch | Built-in security features |
| Organize application code yourself | Structured MVC architecture |

## 3. Development Requirements

Before creating a Laravel project, make sure you have:

| Software | Purpose |
|---|---|
| PHP 8.3 or newer | Backend programming language |
| Composer | PHP dependency manager |
| MySQL | Database |
| WAMP / XAMPP | Local development environment |
| VS Code | Code editor |
| Git | Version control |

Check your installed versions using:

```bash
php -v
composer -V
git --version
```

**Note:** MySQL is not required for this first lesson. We will configure the database in a later lesson.

## 4. Create Your First Laravel Project

We will build a simple Task Management System throughout this course.

Our project will eventually support:

- Creating tasks.
- Viewing tasks.
- Editing tasks.
- Deleting tasks.
- Managing task status.

### Step 1: Open the terminal

Open Command Prompt or PowerShell.

Navigate to your project directory.

Example using WAMP:

```bash
cd C:\wamp64\www\laravel
```

### Step 2: Create a Laravel project

Run:

```bash
composer create-project laravel/laravel task-manager
```

Composer will download Laravel and install its dependencies.

Wait until the installation completes.

### Step 3: Enter the project directory

```bash
cd task-manager
```

### Step 4: Check the Laravel version

```bash
php artisan --version
```

Expected output (version may vary):

```text
Laravel Framework 13.x.x
```

### Step 5: Start the development server

```bash
php artisan serve
```

Expected output:

```text
INFO  Server running on [http://127.0.0.1:8000].
```

Open your browser:

http://127.0.0.1:8000

You should see the Laravel welcome page.

**Congratulations! Your first Laravel application is running.**

## 5. Understand the Artisan Command

Artisan is Laravel's built-in command-line interface (CLI).

It helps developers perform common development tasks without manually creating every file.

For example:

```bash
php artisan list
```

Displays available Artisan commands.

```bash
php artisan about
```

Displays information about your Laravel application and environment.

```bash
php artisan route:list
```

Displays registered application routes.

We will use Artisan throughout this course.

## 6. Initialize Git

Version control helps us track code changes and maintain the history of our project.

We will use Git and GitHub to manage our course project.

### Step 1: Initialize Git

Open another terminal inside the `task-manager` directory and run:

```bash
git init
```

### Step 2: Check project status

```bash
git status
```

Laravel already includes a `.gitignore` file that excludes environment-specific files such as `.env`.

**Important:** Never commit your `.env` file because it may contain database credentials, API keys and other secrets.

### Step 3: Add project files

```bash
git add .
```

### Step 4: Create your first commit

```bash
git commit -m "Initial Laravel 13 project setup"
```

Your project is now under Git version control.

You can connect it to a GitHub repository and push it when ready.

## 7. Practical Assignment

Complete the following tasks independently:

1. Create a Laravel project named `task-manager`.
2. Run the project locally.
3. Open the Laravel welcome page.
4. Check the installed Laravel version.
5. Run `php artisan list`.
6. Run `php artisan about`.
7. Initialize a Git repository.
8. Commit your project.

### Expected Result

At the end of this lesson, you should have:

- A working Laravel 13 project.
- A locally running development server.
- A basic understanding of Artisan.
- A Git repository containing your project.

## 8. Knowledge Check

Answer these questions before moving to the next lesson.

1. What is Laravel?
2. What is the purpose of Composer?
3. What is Artisan?
4. Which command creates a new Laravel project?
5. Which command starts the Laravel development server?
6. Why should we avoid committing the `.env` file?
7. What is the purpose of Git?

## 9. Official Documentation

Read the following documentation for additional understanding:

- [Laravel 13 – Installation](https://laravel.com/docs/13.x/installation)
- [Laravel 13 – Configuration](https://laravel.com/docs/13.x/configuration)
- [Laravel 13 – Artisan Console](https://laravel.com/docs/13.x/artisan)
- [Git Documentation](https://git-scm.com/doc)

---

**Next Lesson: Lesson 02 – Laravel Project Structure**

We will explore Laravel's directory structure, understand important files and folders, and learn how a Laravel application processes a request.