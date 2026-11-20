# My Notes – Laravel 13

A simple notes management application built with **Laravel 13**. This project is part of my Laravel learning journey and demonstrates how to build a practical web application using Laravel's MVC architecture, Eloquent ORM, Blade templating, authentication, CRUD operations, and authorization policies.

## Overview

My Notes allows users to register, log in, and manage their own notes (called entries in the application). It is intended as a hands-on learning project for understanding how the main parts of a Laravel application work together.

## Features

- **User authentication** – user registration and login.
- **Create notes** – add a new note/entry.
- **View notes** – display the notes list and individual note details.
- **Update notes** – edit an existing note.
- **Delete notes** – remove a note.
- **Authorization** – use Laravel policies to control access to protected resources.
- **Reusable Blade components** – share UI elements across views.
- **Database migrations** – manage the database schema through Laravel migrations.

> The application uses the term `Entry` in its model and controller code to represent a note.

## Tech Stack

| Technology | Purpose |
|---|---|
| PHP | Backend programming language |
| Laravel 13 | PHP web application framework |
| MySQL / SQLite | Database (configured through `.env`) |
| Eloquent ORM | Database queries and model interaction |
| Blade | Server-side view templating |
| Vite | Frontend asset development and build |
| CSS / JavaScript | Frontend styling and interactions |
| PHPUnit | Automated testing |

## Requirements

Before installing the project, make sure you have:

- PHP **8.3 or newer**
- Composer
- Node.js and npm
- A database supported by Laravel (for example, MySQL or SQLite)
- Git

Check your installed versions:

```bash
php -v
composer --version
node -v
npm -v
```

## Installation

### 1. Clone the repository

From the root of the learning repository:

```bash
git clone https://github.com/techgovinda1/learn-laravel-13.git
cd learn-laravel-13/project-my-notes
```

### 2. Install PHP dependencies

```bash
composer install
```

### 3. Configure environment variables

Create your local environment file:

```bash
cp .env.example .env
```

On Windows PowerShell, use:

```powershell
Copy-Item .env.example .env
```

Generate the application key:

```bash
php artisan key:generate
```

### 4. Configure the database

Update the database settings in `.env` to match your local database.

Example for MySQL:

```dotenv
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=my_notes
DB_USERNAME=root
DB_PASSWORD=
```

Create the `my_notes` database before running the migrations.

For SQLite, configure `DB_CONNECTION=sqlite` and ensure the SQLite database file exists at the path configured by your application.

### 5. Run database migrations

```bash
php artisan migrate
```

This creates the tables required by the application.

### 6. Install frontend dependencies

```bash
npm install
```

### 7. Start the application

For local development, open two terminal windows.

Terminal 1:

```bash
php artisan serve
```

Terminal 2:

```bash
npm run dev
```

Visit:

```text
http://127.0.0.1:8000
```

Alternatively, if the Composer `dev` script is available in your installed project:

```bash
composer run dev
```

## Application Structure

Important files and directories:

```text
project-my-notes/
├── app/
│   ├── Http/
│   │   └── Controllers/
│   │       ├── AuthController.php
│   │       ├── Controller.php
│   │       └── EntryController.php
│   ├── Models/
│   │   ├── Entry.php
│   │   └── User.php
│   ├── Policies/
│   │   └── UserPolicy.php
│   └── Providers/
│       └── AppServiceProvider.php
├── database/
│   ├── factories/
│   ├── migrations/
│   └── seeders/
├── resources/
│   ├── css/
│   ├── js/
│   └── views/
│       ├── auth/
│       ├── components/
│       └── entries/
├── routes/
│   ├── console.php
│   └── web.php
├── tests/
├── artisan
├── composer.json
└── package.json
```

## Laravel Concepts Practiced

This project provides practical examples of the following concepts:

| Concept | What it is used for |
|---|---|
| Routing | Mapping URLs to application actions |
| Controllers | Handling incoming requests and application logic |
| MVC architecture | Separating models, views, and controllers |
| Eloquent models | Working with database records |
| Migrations | Creating and updating database tables |
| Authentication | Registering and logging in users |
| Authorization policies | Controlling access to application resources |
| Blade templates | Rendering dynamic HTML pages |
| Blade components | Reusing view elements |
| Form handling | Receiving and processing user input |
| Validation | Checking submitted data before processing |
| HTTP methods | Handling GET, POST, PUT/PATCH, and DELETE requests |

## Useful Artisan Commands

```bash
# List registered routes
php artisan route:list

# List available Artisan commands
php artisan list

# Run database migrations
php artisan migrate

# Roll back the latest migration batch
php artisan migrate:rollback

# Clear application caches
php artisan optimize:clear

# Run automated tests
php artisan test
```

## Running Tests

Run the project's automated test suite:

```bash
php artisan test
```

## Learning Goals

The main goal of this project is to understand Laravel by building a working application instead of learning individual concepts only in theory.

By completing and extending this project, I aim to improve my understanding of:

- Laravel application structure and request lifecycle.
- MVC architecture and clean code organization.
- Database design, migrations, and Eloquent ORM.
- Authentication and authorization.
- CRUD workflows and form validation.
- Blade views and reusable components.
- Testing and debugging Laravel applications.

## Future Improvements

Potential enhancements as my Laravel skills develop:

- Add note categories and tags.
- Add search, sorting, and pagination.
- Add note status or pinning functionality.
- Add automated feature tests for authentication and note ownership.
- Improve validation and error handling.
- Add a richer dashboard and responsive UI.

## Repository

This project is one of the practical applications in my Laravel 13 learning repository:

**[Learn Laravel 13 – GitHub Repository](https://github.com/techgovinda1/learn-laravel-13)**

---

**Author:** [Govinda Dahal](https://github.com/techgovinda1)

*Built for learning Laravel 13 through practical application development.*
