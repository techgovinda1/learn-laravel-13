# Lesson 10 – Authentication and Authorization

**Course:** Practical Laravel 13  
**Level:** Beginner  
**Estimated Time:** 60–90 minutes  
**Project:** Task Management System

---

## 1. Learning Objectives

By the end of this lesson, you will understand how to:

- Explain authentication and authorization.
- Set up Laravel authentication using an official starter kit.
- Register a new user.
- Log in and log out.
- Protect routes using the `auth` middleware.
- Access the currently authenticated user.
- Understand how authentication prepares an application for multiple users.

---

## 2. Authentication vs. Authorization

These are two different security concepts.

### Authentication

Authentication answers:

**Who are you?**

It verifies a user's identity.

Examples:
- Registering an account.
- Logging in with email and password.
- Logging out.
- Resetting a password.

### Authorization

Authorization answers:

**What are you allowed to do?**

It checks whether an authenticated user has permission to perform an action.

Examples:
- A user can edit their own tasks.
- An administrator can manage all users.
- A regular user cannot access the admin dashboard.

| Authentication | Authorization |
|---|---|
| Verifies identity | Checks permissions |
| Login and logout | Roles and permissions |
| Who are you? | What can you do? |

---

## 3. How Laravel Handles Authentication

Laravel provides authentication functionality through its authentication system.

Important components:

| Component | Purpose |
|---|---|
| User model | Represents application users |
| `users` table | Stores user information |
| Auth facade | Provides authentication methods |
| `auth` middleware | Protects routes |
| Laravel Fortify | Provides authentication backend functionality |
| Starter kits | Provide ready-to-use authentication interfaces |

Laravel's official starter kits provide registration, login, password reset, and other authentication features.

For this course, we will use the **Livewire starter kit** because it works well with Laravel's PHP and Blade-oriented development approach.

---

## 4. Create an Authentication Practice Application

Laravel 13 starter kits are selected when creating a new application using the Laravel installer.

Since our existing Task Management System was created without an authentication starter kit, we will first create a separate practice application. This avoids replacing or accidentally breaking the existing project.

### Step 1: Install the Laravel installer

If it is not already installed:

```bash
composer global require laravel/installer
```

### Step 2: Create a new application

```bash
laravel new task-manager-auth
```

The installer will ask you to select options.

Choose:

- Starter kit: Livewire
- Authentication: Laravel's built-in authentication option, if prompted
- Database: MySQL
- Testing framework: Your preferred option

Follow the installer prompts.

### Step 3: Enter the project directory

```bash
cd task-manager-auth
```

### Step 4: Install frontend dependencies

```bash
npm install
npm run build
```

### Step 5: Configure the database

Create a database in phpMyAdmin:

```text
task_manager_auth
```

Update your `.env` file:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=task_manager_auth
DB_USERNAME=root
DB_PASSWORD=
```

Use your actual MySQL username and password if they differ.

### Step 6: Run migrations

```bash
php artisan migrate
```

This creates the database tables, including the default `users` table.

### Step 7: Start the application

```bash
composer run dev
```

Open the local URL shown in your terminal.

---

## 5. Explore the Authentication Pages

The Livewire starter kit provides ready-made authentication pages.

Visit:

```text
/register
/login
```

### Practical Exercise A: Register a User

1. Open `/register`.
2. Enter a name.
3. Enter an email address.
4. Enter a password.
5. Confirm the password.
6. Submit the form.

After successful registration, the starter kit will authenticate the new user and redirect them according to its configured flow.

### Practical Exercise B: Log Out

Use the logout option provided by the starter kit.

You should now be logged out.

### Practical Exercise C: Log In

1. Open `/login`.
2. Enter the registered email.
3. Enter the password.
4. Submit the form.

You should be authenticated again.

**Important:** Do not store plain-text passwords. Laravel's authentication system uses password hashing.

---

## 6. Understand the User Table

Open your database in phpMyAdmin.

Find the `users` table.

A default Laravel user table contains fields such as:

| Field | Purpose |
|---|---|
| id | Unique user ID |
| name | User's name |
| email | User's email |
| email_verified_at | Email verification timestamp |
| password | Hashed password |
| remember_token | Remember-me functionality |
| created_at | Registration timestamp |
| updated_at | Last update timestamp |

The password stored in the database is a hash, not the original password.

---

## 7. Protect Routes Using Authentication Middleware

One of Laravel's most useful authentication features is the `auth` middleware.

It prevents unauthenticated visitors from accessing protected routes.

Open:

```text
routes/web.php
```

Add a protected route:

```php
use Illuminate\Support\Facades\Route;

Route::get('/my-account', function () {
    return 'Welcome to your account!';
})->middleware('auth');
```

Now visit:

```text
/my-account
```

### Test the Route

**Test 1: Logged out**

- Log out of the application.
- Visit `/my-account`.
- Laravel should redirect you to the login page.

**Test 2: Logged in**

- Log in again.
- Visit `/my-account`.
- You should see:

```text
Welcome to your account!
```

This is your first protected route.

---

## 8. Protect Multiple Routes

Instead of adding middleware to every route individually, use a route group.

```php
use Illuminate\Support\Facades\Route;

Route::middleware('auth')->group(function () {

    Route::get('/my-account', function () {
        return 'My Account';
    });

    Route::get('/my-tasks', function () {
        return 'My Tasks';
    });

    Route::get('/my-reports', function () {
        return 'My Reports';
    });

});
```

All routes inside this group require authentication.

### Apply This Concept to the Task Management System

When authentication is integrated into the original Task Management System, its task routes can be protected like this:

```php
use App\Http\Controllers\TaskController;
use Illuminate\Support\Facades\Route;

Route::middleware('auth')->group(function () {
    Route::resource('tasks', TaskController::class);
});
```

**Important:** This route protection requires an authentication setup that provides a login route. The separate practice application already has that setup.

---

## 9. Retrieve the Authenticated User

Laravel provides several ways to access the currently logged-in user.

### Using the Auth facade

```php
use Illuminate\Support\Facades\Auth;

$user = Auth::user();

echo $user->name;
```

Retrieve the user's ID:

```php
$userId = Auth::id();
```

Check whether a user is logged in:

```php
if (Auth::check()) {
    echo 'User is logged in';
} else {
    echo 'Guest user';
}
```

### Using the Request object

Inside a controller:

```php
use Illuminate\Http\Request;

public function profile(Request $request)
{
    $user = $request->user();

    return $user->name;
}
```

### Practical Exercise D: Display the Current User

Add a route to `routes/web.php`:

```php
use Illuminate\Support\Facades\Auth;

Route::get('/my-profile', function () {

    $user = Auth::user();

    return 'Welcome, ' . $user->name;

})->middleware('auth');
```

Log in and visit:

```text
/my-profile
```

You should see the name of the authenticated user.

---

## 10. Authentication vs. Task Ownership

Our Task Management System currently allows users to create, view, update, and delete tasks.

But consider this situation:

- User A creates a task.
- User B logs in.
- User B tries to access User A's task.

Should User B be allowed to edit or delete it?

**No.**

Authentication alone does not solve this problem.

We also need authorization and task ownership.

In a future lesson, we will connect tasks to users using a database relationship:

```text
User
 └── Tasks
      ├── Task 1
      ├── Task 2
      └── Task 3
```

This will allow us to build a multi-user Task Management System where users manage their own tasks.

---

## 11. Authentication and Authorization Summary

| Feature | Laravel concept |
|---|---|
| Register | Starter kit / Fortify |
| Login | Authentication |
| Logout | Authentication |
| Check logged-in user | `Auth::check()` |
| Get current user | `Auth::user()` |
| Get current user ID | `Auth::id()` |
| Protect routes | `auth` middleware |
| Check permissions | Gates and Policies |
| Restrict task ownership | Authorization and relationships |

Remember:

- Authentication verifies identity.
- Authorization checks permissions.
- Middleware protects routes.
- Gates and Policies help control access to specific actions.

---

## 12. Assignment

Complete the following tasks:

### Task 1: Authentication

- [ ] Create the authentication practice application.
- [ ] Register two different users.
- [ ] Log in with each account.
- [ ] Test logout functionality.

### Task 2: Protected Routes

Create these routes:

```text
/dashboard
/profile
/my-tasks
```

Protect all three routes using the `auth` middleware.

### Task 3: Display User Information

Create a profile route that displays:

- User name
- User email
- User ID

Use the `Auth` facade.

### Task 4: Test Security

- [ ] Access protected routes while logged out.
- [ ] Verify that Laravel redirects you to login.
- [ ] Log in and access the routes again.
- [ ] Confirm that the authenticated user information is displayed correctly.

---

## 13. Knowledge Check

1. What is authentication?
2. What is authorization?
3. What is the purpose of the `auth` middleware?
4. Which Laravel facade provides authentication methods?
5. How do you retrieve the currently authenticated user?
6. Why should passwords be hashed?
7. Does authentication automatically prevent users from editing another user's tasks?

---

## 14. Completion Checklist

You should now be able to:

- [ ] Explain authentication and authorization.
- [ ] Set up Laravel authentication using a starter kit.
- [ ] Register and log in users.
- [ ] Log out users.
- [ ] Protect routes using middleware.
- [ ] Retrieve authenticated user information.
- [ ] Explain why task ownership requires authorization.

---

## 15. Official Laravel Documentation

- [Laravel 13 – Authentication](https://laravel.com/docs/13.x/authentication)
- [Laravel 13 – Starter Kits](https://laravel.com/docs/13.x/starter-kits)
- [Laravel 13 – Authorization](https://laravel.com/docs/13.x/authorization)
- [Laravel 13 – Middleware](https://laravel.com/docs/13.x/middleware)

---

## Next Lesson

**Lesson 11 – Eloquent Relationships**

Topics:

- Understanding database relationships.
- One-to-one relationships.
- One-to-many relationships.
- Many-to-many relationships.
- `hasMany()` and `belongsTo()`.
- Connecting users with their tasks.

**Upcoming practical goal:** Each registered user will be able to manage their own tasks.