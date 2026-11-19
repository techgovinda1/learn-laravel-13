# Lesson 11 – Eloquent Relationships

**Course:** Practical Laravel 13  
**Level:** Beginner  
**Estimated Time:** 60–90 minutes  
**Project:** Task Management System

---

## 1. Learning Objectives

By the end of this lesson, you will understand how to:

- Explain database relationships.
- Understand one-to-one, one-to-many, and many-to-many relationships.
- Define relationships using Eloquent.
- Use `hasMany()` and `belongsTo()`.
- Connect users with their tasks.
- Retrieve related records.
- Create records through relationships.
- Ensure users can access only their own tasks.

---

## 2. What Are Eloquent Relationships?

In a real application, database tables are usually connected.

For example, consider our Task Management System:

- One user can create multiple tasks.
- Each task belongs to one user.
- A task may have multiple categories.
- A category may contain multiple tasks.

Instead of manually writing SQL queries to connect these tables, Laravel provides **Eloquent Relationships**.

Eloquent relationships allow models to interact with related database records.

---

## 3. Types of Database Relationships

Laravel supports several relationship types. The three basic relationships are:

### A. One-to-One

One record is associated with one record in another table.

Example:

```text
users
  |
  | One
  |
  v
profiles
```

One user has one profile.

Laravel relationship methods:

```php
hasOne()
belongsTo()
```

### B. One-to-Many

One record can be associated with multiple records.

Example:

```text
users
  |
  | One
  |
  +------ Task 1
  |
  +------ Task 2
  |
  +------ Task 3
```

One user can have many tasks, but each task belongs to one user.

Laravel relationship methods:

```php
hasMany()
belongsTo()
```

### C. Many-to-Many

Multiple records can be associated with multiple records.

Example:

```text
tasks                 categories

Task 1  ------------  Work
Task 1  ------------  Personal
Task 2  ------------  Work
Task 3  ------------  Personal
```

A task can have multiple categories, and a category can belong to multiple tasks.

Laravel relationship method:

```php
belongsToMany()
```

---

## 4. Practical Project: Connect Users and Tasks

We will implement a one-to-many relationship.

**Requirement:**

- One user can create many tasks.
- Each task belongs to one user.
- Users should see only their own tasks.

Our database structure will look like this:

```text
users
-----
id
name
email
password

tasks
-----
id
user_id
title
description
status
priority
due_date
created_at
updated_at
```

The `user_id` column connects each task to its owner.

**Important:** This lesson assumes you have completed Lesson 10 and have authentication installed in your Task Management System.

---

## 5. Step 1 – Add user_id to the Tasks Table

We need a foreign key in the `tasks` table.

Run:

```bash
php artisan make:migration add_user_id_to_tasks_table --table=tasks
```

Open the newly created migration file in:

```text
database/migrations/
```

Add:

```php
public function up(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->foreignId('user_id')
            ->nullable()
            ->constrained()
            ->nullOnDelete();
    });
}

public function down(): void
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->dropForeign(['user_id']);
        $table->dropColumn('user_id');
    });
}
```

### Why use nullable()?

Our existing tasks may not have an associated user.

Making `user_id` nullable allows the migration to run without requiring us to assign an owner to every existing task immediately.

### Why use constrained()?

It creates a foreign key constraint referencing the `id` column in the `users` table.

### Why use nullOnDelete()?

If a user is deleted, the associated tasks will remain, but their `user_id` will become `NULL`.

Run the migration:

```bash
php artisan migrate
```

Open phpMyAdmin and check the `tasks` table.

You should now see:

```text
user_id
```

---

## 6. Step 2 – Define the User Relationship

Open:

```text
app/Models/User.php
```

Add the `tasks()` relationship.

```php
use App\Models\Task;
use Illuminate\Database\Eloquent\Relations\HasMany;

public function tasks(): HasMany
{
    return $this->hasMany(Task::class);
}
```

Your `User` model will now have a relationship with the `Task` model.

The method:

```php
$user->tasks()
```

returns a relationship query that can retrieve or create tasks belonging to that user.

---

## 7. Step 3 – Define the Task Relationship

Open:

```text
app/Models/Task.php
```

Add the `user()` relationship.

```php
use App\Models\User;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

public function user(): BelongsTo
{
    return $this->belongsTo(User::class);
}
```

Now a task can access its owner.

For example:

```php
$task->user;
```

This retrieves the user associated with that task.

---

## 8. Step 4 – Understand the Relationship

We now have two relationship methods.

In the User model:

```php
public function tasks(): HasMany
{
    return $this->hasMany(Task::class);
}
```

In the Task model:

```php
public function user(): BelongsTo
{
    return $this->belongsTo(User::class);
}
```

Remember:

| Model | Method | Meaning |
|---|---|---|
| User | `tasks()` | A user has many tasks |
| Task | `user()` | A task belongs to a user |

The foreign key is stored in the `tasks` table:

```text
tasks.user_id
```

---

## 9. Step 5 – Allow user_id in Task Creation

Open:

```text
app/Models/Task.php
```

Update the `$fillable` property:

```php
protected $fillable = [
    'title',
    'description',
    'status',
    'priority',
    'due_date',
    'category',
    'user_id',
];
```

This allows Eloquent to assign `user_id` when creating a task through the relationship.

**Security note:** Do not accept `user_id` directly from an ordinary user's form. The authenticated user's relationship should determine the owner.

---

## 10. Step 6 – Create Tasks Through the Relationship

Previously, we created tasks using:

```php
Task::create($validated);
```

Now we want the logged-in user to own the task automatically.

Inside your `TaskController`, update the `store()` method:

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => 'required|string|max:255',
        'description' => 'nullable|string',
        'status' => 'required|string',
        'priority' => 'nullable|string',
        'due_date' => 'nullable|date',
    ]);

    $request->user()->tasks()->create($validated);

    return redirect()
        ->route('tasks.index')
        ->with('success', 'Task created successfully.');
}
```

Notice:

```php
$request->user()->tasks()->create($validated);
```

Laravel automatically assigns the authenticated user's ID to `user_id`.

You do not need to manually add the user's ID to the form.

---

## 11. Step 7 – Retrieve Only the Logged-in User's Tasks

Previously, we retrieved all tasks:

```php
$tasks = Task::latest()->get();
```

This would show tasks from every user.

Instead, update the `index()` method:

```php
public function index(Request $request)
{
    $tasks = $request->user()
        ->tasks()
        ->latest()
        ->get();

    return view('tasks.index', compact('tasks'));
}
```

Now the application retrieves only tasks belonging to the logged-in user.

For example:

```text
User A
  - Task 1
  - Task 2

User B
  - Task 3
  - Task 4
```

User A will see Task 1 and Task 2.

User B will see Task 3 and Task 4.

---

## 12. Step 8 – Protect Individual Tasks

Protecting the task routes with authentication is not enough.

A logged-in user could try changing the task ID in the URL to access another user's task.

For example:

```text
/tasks/10/edit
```

We must verify task ownership inside the controller.

A simple approach is to retrieve the task through the authenticated user's relationship.

### Update show()

```php
public function show(Request $request, Task $task)
{
    $task = $request->user()
        ->tasks()
        ->findOrFail($task->id);

    return view('tasks.show', compact('task'));
}
```

### Update edit()

```php
public function edit(Request $request, Task $task)
{
    $task = $request->user()
        ->tasks()
        ->findOrFail($task->id);

    return view('tasks.edit', compact('task'));
}
```

### Update update()

```php
public function update(Request $request, Task $task)
{
    $task = $request->user()
        ->tasks()
        ->findOrFail($task->id);

    $validated = $request->validate([
        'title' => 'required|string|max:255',
        'description' => 'nullable|string',
        'status' => 'required|string',
        'priority' => 'nullable|string',
        'due_date' => 'nullable|date',
    ]);

    $task->update($validated);

    return redirect()
        ->route('tasks.index')
        ->with('success', 'Task updated successfully.');
}
```

### Update destroy()

```php
public function destroy(Request $request, Task $task)
{
    $task = $request->user()
        ->tasks()
        ->findOrFail($task->id);

    $task->delete();

    return redirect()
        ->route('tasks.index')
        ->with('success', 'Task deleted successfully.');
}
```

**Important:** `findOrFail()` returns a 404 response if the task does not belong to the authenticated user. This prevents users from accessing or modifying another user's task through these controller actions.

---

## 13. Step 9 – Retrieve the Task Owner

Because we defined the `user()` relationship in the Task model, we can retrieve the task owner.

Example:

```php
$task = Task::find(1);

echo $task->user->name;
```

In a Blade file:

```blade
<p>Task Owner: {{ $task->user?->name ?? 'Unassigned' }}</p>
```

The null-safe operator is useful because existing tasks may have a `NULL` user ID.

---

## 14. Step 10 – Test Relationships Using Tinker

Laravel Tinker allows us to interact with our application models directly.

Run:

```bash
php artisan tinker
```

### Find a User

```php
$user = App\Models\User::first();
```

### Retrieve the User's Tasks

```php
$user->tasks;
```

### Count the User's Tasks

```php
$user->tasks()->count();
```

### Create a Task for the User

```php
$user->tasks()->create([
    'title' => 'Learn Laravel Relationships',
    'description' => 'Practice hasMany and belongsTo',
    'status' => 'Pending',
]);
```

### Find a Task

```php
$task = App\Models\Task::first();
```

### Retrieve the Task Owner

```php
$task->user;
```

### Retrieve the Owner's Name

```php
$task->user?->name;
```

Exit Tinker:

```php
exit
```

---

## 15. Relationship Methods vs. Properties

This is an important concept.

Consider:

```php
$user->tasks
```

This accesses the relationship as a property and retrieves the related tasks.

Now consider:

```php
$user->tasks()
```

This returns the relationship query.

You can use it to apply additional conditions:

```php
$user->tasks()
    ->where('status', 'Pending')
    ->get();
```

Count tasks:

```php
$user->tasks()->count();
```

Create a task:

```php
$user->tasks()->create([
    'title' => 'Practice Laravel',
    'status' => 'Pending',
]);
```

Remember:

- `tasks` accesses the related data.
- `tasks()` gives you the relationship query.

---

## 16. Introduction to Other Relationships

We have implemented one-to-many. Here are the other two basic types.

### One-to-One Example

A user has one profile.

In `User.php`:

```php
public function profile(): HasOne
{
    return $this->hasOne(Profile::class);
}
```

In `Profile.php`:

```php
public function user(): BelongsTo
{
    return $this->belongsTo(User::class);
}
```

The `profiles` table would contain:

```text
id
user_id
bio
created_at
updated_at
```

### Many-to-Many Example

A task can have multiple labels, and a label can belong to multiple tasks.

In `Task.php`:

```php
public function labels(): BelongsToMany
{
    return $this->belongsToMany(Label::class);
}
```

In `Label.php`:

```php
public function tasks(): BelongsToMany
{
    return $this->belongsToMany(Task::class);
}
```

This relationship uses an intermediate table:

```text
label_task
----------
id
label_id
task_id
```

Laravel uses the `belongsToMany()` method for both sides.

**Note:** These examples illustrate the relationship definitions. The models, migrations, and imports must also be created before using them.

---

## 17. Assignment

Complete the following tasks in your Task Management System.

### Task 1: Database

- [ ] Add `user_id` to the tasks table.
- [ ] Run the migration.
- [ ] Verify the foreign key in phpMyAdmin.

### Task 2: Models

- [ ] Add `tasks()` to the User model.
- [ ] Add `user()` to the Task model.
- [ ] Add `user_id` to the Task model's `$fillable` property.

### Task 3: Controller

- [ ] Create tasks through the authenticated user's relationship.
- [ ] Display only the logged-in user's tasks.
- [ ] Protect individual task operations using ownership checks.

### Task 4: Testing

- [ ] Register User A.
- [ ] Create two tasks as User A.
- [ ] Log out.
- [ ] Register User B.
- [ ] Create two different tasks as User B.
- [ ] Confirm User B cannot see User A's tasks.
- [ ] Try accessing User A's task URL while logged in as User B.
- [ ] Confirm the application returns a 404 response.

---

## 18. Knowledge Check

1. What is an Eloquent relationship?
2. What is the difference between `hasMany()` and `belongsTo()`?
3. Where is the foreign key stored in a one-to-many relationship?
4. Why do we use `user_id` in the tasks table?
5. What is the difference between `$user->tasks` and `$user->tasks()`?
6. How can we create a task through a user's relationship?
7. Why is authentication alone insufficient to protect individual tasks?
8. What is a many-to-many relationship?

---

## 19. Completion Checklist

You should now be able to:

- [ ] Explain one-to-one relationships.
- [ ] Explain one-to-many relationships.
- [ ] Explain many-to-many relationships.
- [ ] Define `hasMany()` relationships.
- [ ] Define `belongsTo()` relationships.
- [ ] Create related records using Eloquent.
- [ ] Retrieve related records.
- [ ] Connect users and tasks.
- [ ] Restrict task access based on ownership.

---

## 20. Official Laravel Documentation

- [Laravel 13 – Eloquent Relationships](https://laravel.com/docs/13.x/eloquent-relationships)
- [Laravel 13 – Eloquent ORM](https://laravel.com/docs/13.x/eloquent)
- [Laravel 13 – Migrations](https://laravel.com/docs/13.x/migrations)
- [Laravel 13 – Authorization](https://laravel.com/docs/13.x/authorization)

---

## Next Lesson

**Lesson 12 – Middleware and Access Control**

Topics:

- Understanding middleware.
- Creating custom middleware.
- Protecting routes.
- Redirecting users based on conditions.
- Understanding route middleware groups.
- Applying middleware to the Task Management System.

**Upcoming practical goal:** Build a more secure application with controlled access to different sections.