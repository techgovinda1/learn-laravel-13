# Lesson 06: Database Configuration and Migrations

**Course:** Learn Laravel 13 – Practical Beginner Course  
**Level:** Beginner  
**Duration:** 45–60 minutes  
**Project:** Task Management System

---

## 1. Learning Objectives

By the end of this lesson, you will be able to:

- Understand how Laravel connects to a database.
- Configure MySQL in Laravel.
- Understand database migrations.
- Create database tables using migrations.
- Run and roll back migrations.
- Use common Artisan migration commands.
- Understand migration files and their structure.

---

## 2. What is a Database?

A database stores application information permanently.

For our Task Management System, we need to store information such as:

| Task ID | Title | Status |
|---|---|---|
| 1 | Learn Laravel | Completed |
| 2 | Build Task Manager | In Progress |
| 3 | Learn MySQL | Pending |

Instead of keeping this information in PHP arrays, we will store it in a MySQL database.

### What is a migration?

A migration is a PHP file that defines changes to the database structure.

For example, a migration can:

- Create a table.
- Add a column.
- Modify a table.
- Remove a table.

Migrations allow developers to manage database structures through code instead of manually creating every table.

---

## 3. Configure MySQL Database

We will use MySQL through WAMP.

### Step 1: Start WAMP

Make sure the following services are running:

- Apache
- MySQL

Open phpMyAdmin:

```text
http://localhost/phpmyadmin
```

### Step 2: Create a database

Create a database named:

```text
task_manager
```

Use the `utf8mb4_unicode_ci` collation if available.

You do not need to create any tables manually. Laravel migrations will create them.

### Step 3: Configure the `.env` file

Open the `.env` file in your Laravel project.

Find the database configuration and update it:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=task_manager
DB_USERNAME=root
DB_PASSWORD=
```

**Note:** The example assumes your local WAMP MySQL uses the `root` account without a password. Use your actual MySQL username, password, and port if they are different.

### Step 4: Clear configuration cache

Run:

```bash
php artisan config:clear
```

This ensures Laravel reads the updated configuration.

---

## 4. Understanding Migration Files

Laravel stores migration files inside:

```text
database/migrations/
```

When you create a new Laravel project, it already includes migrations for some default tables, such as:

- users
- cache
- jobs

Migration filenames usually contain a timestamp.

Example:

```text
2026_11_19_000001_create_tasks_table.php
```

The timestamp determines the order in which migrations run.

### Migration structure

A migration generally contains two methods:

```php
public function up(): void
{
    // Apply database changes
}

public function down(): void
{
    // Reverse database changes
}
```

| Method | Purpose |
|---|---|
| `up()` | Creates or modifies database structures |
| `down()` | Reverses the changes |

---

## 5. Create Your First Migration

Let's create a `tasks` table.

Run:

```bash
php artisan make:migration create_tasks_table
```

Laravel creates a migration file inside:

```text
database/migrations/
```

Open the newly generated migration file.

Replace its contents with:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('tasks', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('description')->nullable();
            $table->string('status')->default('Pending');
            $table->date('due_date')->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('tasks');
    }
};
```

### Understanding the columns

| Column | Purpose |
|---|---|
| `id` | Unique task identifier |
| `title` | Task title |
| `description` | Task details; optional |
| `status` | Task status; defaults to Pending |
| `due_date` | Task deadline; optional |
| `created_at` | Record creation time |
| `updated_at` | Last update time |

### Understanding common column methods

| Method | Purpose |
|---|---|
| `id()` | Creates an auto-incrementing primary key |
| `string()` | Creates a string column |
| `text()` | Creates a text column |
| `integer()` | Creates an integer column |
| `boolean()` | Creates a true/false column |
| `date()` | Creates a date column |
| `decimal()` | Creates a decimal column |
| `nullable()` | Allows NULL values |
| `default()` | Defines a default value |
| `timestamps()` | Creates `created_at` and `updated_at` |

---

## 6. Run Migrations

Before running the migration, make sure your MySQL database is configured correctly.

Run:

```bash
php artisan migrate
```

Expected output will show Laravel running pending migrations.

Open phpMyAdmin and select:

```text
task_manager
```

You should now see the `tasks` table, along with Laravel's default tables.

The `tasks` table should contain:

```text
id
title
description
status
due_date
created_at
updated_at
```

Laravel also creates a `migrations` table to track which migrations have already run.

---

## 7. Understand Migration Commands

Laravel provides several useful migration commands.

### Run pending migrations

```bash
php artisan migrate
```

### Check migration status

```bash
php artisan migrate:status
```

### Roll back the latest migration batch

```bash
php artisan migrate:rollback
```

### Roll back the latest batch and migrate again

```bash
php artisan migrate:refresh
```

### Drop all tables and run migrations again

```bash
php artisan migrate:fresh
```

**Important:** `migrate:fresh` drops all tables before recreating them. It deletes existing database data. Do not use it on a production database or any database containing data you need to keep.

---

## 8. Add a New Column to an Existing Table

Imagine that we want to add a `priority` column to our tasks table.

Do not edit an already executed migration just to add a new column.

Instead, create another migration:

```bash
php artisan make:migration add_priority_to_tasks_table
```

Open the generated migration file:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('tasks', function (Blueprint $table) {
            $table->string('priority')->default('Normal');
        });
    }

    public function down(): void
    {
        Schema::table('tasks', function (Blueprint $table) {
            $table->dropColumn('priority');
        });
    }
};
```

Run:

```bash
php artisan migrate
```

The `priority` column will now be added to the existing `tasks` table.

---

## 9. Practical Assignment

Create a database structure for the Task Management System.

### Tasks

1. Create a MySQL database named `task_manager`.
2. Configure the database connection in `.env`.
3. Create a `tasks` migration.
4. Add the following columns:

| Column | Data Type | Requirement |
|---|---|---|
| id | Big integer | Primary key |
| title | String | Required |
| description | Text | Optional |
| status | String | Default: Pending |
| priority | String | Default: Normal |
| due_date | Date | Optional |
| timestamps | Timestamps | Required |

5. Run the migration.
6. Check the table in phpMyAdmin.
7. Check migration status using Artisan.
8. Create another migration to add a `category` column.

### Expected Result

Your `tasks` table should have:

```text
id
title
description
status
priority
due_date
category
created_at
updated_at
```

---

## 10. Knowledge Check

1. What is a database migration?
2. Where are migration files stored?
3. What is the purpose of the `up()` method?
4. What is the purpose of the `down()` method?
5. Which command creates a migration?
6. Which command executes pending migrations?
7. What does `nullable()` do?
8. What does `timestamps()` create?
9. Why should we create a new migration when adding a column to an existing table?
10. What is the difference between `migrate:rollback` and `migrate:fresh`?

---

## 11. Completion Checklist

- [ ] Started MySQL using WAMP.
- [ ] Created the `task_manager` database.
- [ ] Configured the `.env` database settings.
- [ ] Created a migration for the tasks table.
- [ ] Added the required columns.
- [ ] Executed migrations successfully.
- [ ] Checked the table in phpMyAdmin.
- [ ] Checked migration status.
- [ ] Created a second migration to add a column.
- [ ] Practiced rollback commands.

---

## 12. Official Laravel Documentation

- [Database: Getting Started](https://laravel.com/docs/13.x/database)
- [Database Migrations](https://laravel.com/docs/13.x/migrations)
- [Laravel Configuration](https://laravel.com/docs/13.x/configuration)

---

## Next Lesson: Lesson 07 – Models and Eloquent ORM

In the next lesson, we will learn:

- What is an Eloquent model?
- Creating models using Artisan.
- Understanding model conventions.
- Connecting models to database tables.
- Retrieving database records.
- Creating, updating, and deleting records using Eloquent.

We will connect the Task Management System to the database instead of using sample PHP arrays.