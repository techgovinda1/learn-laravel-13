# Lesson 09: Form Validation and Error Handling

**Course:** Learn Laravel 13 – Practical Beginner Course  
**Level:** Beginner  
**Duration:** 45–60 minutes  
**Project:** Task Management System

---

## 1. Learning Objectives

By the end of this lesson, you will be able to:

- Understand why form validation is important.
- Validate form data using Laravel.
- Use common validation rules.
- Write custom validation messages.
- Display validation errors in Blade.
- Preserve submitted values after validation failure.
- Validate data during create and update operations.
- Understand Laravel's Form Request validation.

---

## 2. What is Form Validation?

Form validation checks whether submitted data meets the application's requirements before it is processed or saved.

For example, when creating a task:

- The title must not be empty.
- The title must not exceed 255 characters.
- The status must be valid.
- The due date must be a valid date.

Without validation, users could submit incomplete or invalid data.

### Example

Suppose a user submits this form:

```text
Title: (empty)
Status: Unknown
Priority: Very Important
Due Date: abc123
```

We should reject this submission and show useful error messages.

Laravel provides built-in validation tools to handle this.

---

## 3. Laravel Validation Using `$request->validate()`

Open:

```text
app/Http/Controllers/TaskController.php
```

Find the `store()` method.

Use the following validation:

```php
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => 'required|string|max:255',
        'description' => 'nullable|string',
        'status' => 'required|in:Pending,In Progress,Completed',
        'priority' => 'required|in:Low,Normal,High',
        'due_date' => 'nullable|date',
    ]);

    Task::create($validated);

    return redirect()
        ->route('tasks.index')
        ->with('success', 'Task created successfully.');
}
```

### Understanding the validation rules

| Rule | Purpose |
|---|---|
| `required` | Field must be provided and not empty |
| `nullable` | Field may contain a null value |
| `string` | Value must be a string |
| `max:255` | Maximum length is 255 characters |
| `min:3` | Minimum length is 3 characters |
| `numeric` | Value must be numeric |
| `integer` | Value must be an integer |
| `email` | Value must be a valid email address |
| `date` | Value must be a valid date |
| `in:a,b,c` | Value must match one of the specified options |
| `confirmed` | Value must have a matching confirmation field |
| `unique:table,column` | Value must be unique in the specified database column |
| `exists:table,column` | Value must exist in the specified database column |

### How validation works

When validation succeeds:

```php
$validated = $request->validate([...]);
```

Laravel returns the validated data.

When validation fails:

1. Laravel redirects back to the previous page.
2. Validation errors are made available to the Blade view.
3. Previously submitted input is flashed to the session.
4. The controller does not continue to the database operation.

You do not need to manually write redirect logic for basic validation failures.

---

## 4. Use an Array for Validation Rules

You can write validation rules as strings or arrays.

**String format:**

```php
$request->validate([
    'title' => 'required|string|max:255',
]);
```

**Array format:**

```php
$request->validate([
    'title' => [
        'required',
        'string',
        'max:255',
    ],
]);
```

Both formats work.

The array format is often easier to read when a field has many validation rules.

---

## 5. Add Custom Validation Messages

Laravel provides default validation messages.

For example:

```text
The title field is required.
```

You can customize these messages.

Update the `store()` method:

```php
public function store(Request $request)
{
    $validated = $request->validate(
        [
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'status' => 'required|in:Pending,In Progress,Completed',
            'priority' => 'required|in:Low,Normal,High',
            'due_date' => 'nullable|date',
        ],
        [
            'title.required' => 'Please enter a task title.',
            'title.max' => 'The task title cannot exceed 255 characters.',
            'status.required' => 'Please select a task status.',
            'priority.required' => 'Please select a priority.',
            'due_date.date' => 'Please enter a valid due date.',
        ]
    );

    Task::create($validated);

    return redirect()
        ->route('tasks.index')
        ->with('success', 'Task created successfully.');
}
```

### Custom message format

```php
'field.rule' => 'Your custom message'
```

Examples:

```php
'title.required' => 'Task title is required.',
'title.max' => 'Task title is too long.',
'status.required' => 'Please select a status.',
```

---

## 6. Display Validation Errors in Blade

Laravel makes validation errors available through the `$errors` variable.

Open:

```text
resources/views/tasks/_form.blade.php
```

Add an error message below each input.

### Title field

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
        <p class="error">{{ $message }}</p>
    @enderror
</div>
```

### Description field

```php
<div>
    <label for="description">Description</label>

    <textarea
        id="description"
        name="description"
    >{{ old('description', $task?->description) }}</textarea>

    @error('description')
        <p class="error">{{ $message }}</p>
    @enderror
</div>
```

### Status field

```php
<div>
    <label for="status">Status</label>

    <select id="status" name="status" required>
        <option value="">Select Status</option>

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
        <p class="error">{{ $message }}</p>
    @enderror
</div>
```

Apply the same pattern to the priority and due date fields.

### Display all validation errors together

You can also display all errors at the top of a form:

```php
@if ($errors->any())
    <div class="validation-errors">
        <h3>Please correct the following errors:</h3>

        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

This is useful when a form contains many fields.

---

## 7. Preserve Submitted Values Using `old()`

Imagine that a user fills out the form but forgets to enter the title.

After validation fails, the form is displayed again.

Without old input handling, the user might need to enter all the information again.

Laravel provides the `old()` helper to preserve previously submitted values.

Example:

```php
<input
    type="text"
    name="title"
    value="{{ old('title') }}"
>
```

For the create and edit forms, we can combine old input with existing task data:

```php
value="{{ old('title', $task?->title) }}"
```

This means:

1. Display the previously submitted value if available.
2. Otherwise, display the existing task title.
3. If neither exists, display an empty value.

The same approach works with textareas and select fields.

---

## 8. Validate the Update Operation

Validation must also be applied when updating existing records.

Open:

```text
app/Http/Controllers/TaskController.php
```

Update the `update()` method:

```php
public function update(Request $request, Task $task)
{
    $validated = $request->validate(
        [
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'status' => 'required|in:Pending,In Progress,Completed',
            'priority' => 'required|in:Low,Normal,High',
            'due_date' => 'nullable|date',
        ],
        [
            'title.required' => 'Please enter a task title.',
            'status.required' => 'Please select a task status.',
            'priority.required' => 'Please select a priority.',
        ]
    );

    $task->update($validated);

    return redirect()
        ->route('tasks.index')
        ->with('success', 'Task updated successfully.');
}
```

Now both create and update operations validate submitted data before saving it.

---

## 9. Validate Data Using Rule Objects

Laravel also supports validation rule objects.

For example, you can use `Rule::in()` to restrict a field to specific values.

At the top of your controller, import:

```php
use Illuminate\Validation\Rule;
```

Then write:

```php
$validated = $request->validate([
    'title' => ['required', 'string', 'max:255'],

    'status' => [
        'required',
        Rule::in(['Pending', 'In Progress', 'Completed']),
    ],

    'priority' => [
        'required',
        Rule::in(['Low', 'Normal', 'High']),
    ],
]);
```

This approach is useful when validation rules become more complex.

---

## 10. Common Validation Examples

### Validate an email address

```php
$request->validate([
    'email' => 'required|email',
]);
```

### Validate a numeric value

```php
$request->validate([
    'amount' => 'required|numeric|min:1',
]);
```

### Validate a password with confirmation

```php
$request->validate([
    'password' => 'required|min:8|confirmed',
]);
```

The form must include:

```html
<input type="password" name="password">

<input type="password" name="password_confirmation">
```

### Validate a unique email address

```php
$request->validate([
    'email' => 'required|email|unique:users,email',
]);
```

This checks that the email does not already exist in the `users` table.

### Validate an existing database record

```php
$request->validate([
    'user_id' => 'required|exists:users,id',
]);
```

This checks that the submitted user ID exists in the `users` table.

---

## 11. Add Basic Error Styling

Open:

```text
public/css/style.css
```

Add:

```css
.error {
    color: #d93025;
    font-size: 14px;
    margin-top: 5px;
}

.validation-errors {
    background: #fff0f0;
    border: 1px solid #d93025;
    padding: 15px;
    margin-bottom: 20px;
}

.validation-errors ul {
    margin-bottom: 0;
}

input,
textarea,
select {
    display: block;
    width: 100%;
    max-width: 500px;
    padding: 10px;
    margin: 8px 0 12px;
    box-sizing: border-box;
}
```

Now validation messages will be easier to identify.

---

## 12. Understand Form Request Validation

As applications grow, controller methods can become large.

Laravel provides Form Request classes to organize validation separately from controller logic.

Create a request class:

```bash
php artisan make:request StoreTaskRequest
```

Laravel creates:

```text
app/Http/Requests/StoreTaskRequest.php
```

Open the file and update it:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreTaskRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'status' => 'required|in:Pending,In Progress,Completed',
            'priority' => 'required|in:Low,Normal,High',
            'due_date' => 'nullable|date',
        ];
    }

    public function messages(): array
    {
        return [
            'title.required' => 'Please enter a task title.',
            'status.required' => 'Please select a task status.',
            'priority.required' => 'Please select a priority.',
        ];
    }
}
```

### Use the Form Request in the controller

Import the request class:

```php
use App\Http\Requests\StoreTaskRequest;
```

Update the `store()` method:

```php
public function store(StoreTaskRequest $request)
{
    Task::create($request->validated());

    return redirect()
        ->route('tasks.index')
        ->with('success', 'Task created successfully.');
}
```

Laravel automatically validates the request before executing the controller method.

If validation fails, Laravel redirects back with errors and old input.

### Why use Form Requests?

- Keep controllers cleaner.
- Organize validation rules in separate files.
- Reuse validation logic.
- Keep custom messages organized.
- Add authorization logic when needed.

**Note:** `authorize()` controls whether the user is allowed to perform the requested action. Returning `true` allows the request for this beginner exercise. Real applications should implement appropriate authorization checks.

---

## 13. Practical Assignment

Improve the Task Management System forms.

### Requirements

1. Make the task title required.
2. Limit the title to 255 characters.
3. Make the description optional.
4. Restrict status to Pending, In Progress, or Completed.
5. Restrict priority to Low, Normal, or High.
6. Validate the due date.
7. Add custom validation messages.
8. Display errors below the relevant fields.
9. Display all errors at the top of the form.
10. Preserve submitted values after validation failure.
11. Apply validation to both create and update operations.
12. Create a `StoreTaskRequest` Form Request class.

### Test Cases

| Test | Expected Result |
|---|---|
| Submit an empty title | Validation error |
| Enter a title longer than 255 characters | Validation error |
| Enter an invalid status | Validation error |
| Enter an invalid priority | Validation error |
| Enter an invalid date | Validation error |
| Submit valid task information | Task saved |
| Submit invalid edit information | Existing task remains unchanged |

---

## 14. Common Problems and Solutions

### Problem 1: Validation errors are not displayed

Make sure your Blade view uses:

```php
@error('title')
    <p>{{ $message }}</p>
@enderror
```

Also confirm that the form submits to the correct route.

### Problem 2: Old input is not displayed

Use:

```php
value="{{ old('title') }}"
```

For edit forms:

```php
value="{{ old('title', $task?->title) }}"
```

### Problem 3: Mass assignment exception

Check the `$fillable` property in:

```text
app/Models/Task.php
```

Make sure the validated fields are included.

### Problem 4: Form Request authorization error

Check the `authorize()` method in the Form Request class.

For this beginner exercise:

```php
public function authorize(): bool
{
    return true;
}
```

In a real application, authorization should be based on the user's permissions.

---

## 15. Knowledge Check

1. What is form validation?
2. Why is validation important?
3. What does the `required` rule do?
4. What is the purpose of `nullable`?
5. What does `$request->validate()` return?
6. How do you display a validation error in Blade?
7. What is the purpose of the `old()` helper?
8. How can you define custom validation messages?
9. What is a Form Request class?
10. What is the purpose of the `authorize()` method?

---

## 16. Completion Checklist

- [ ] Understand Laravel validation.
- [ ] Used common validation rules.
- [ ] Validated task creation.
- [ ] Validated task updates.
- [ ] Added custom validation messages.
- [ ] Displayed field-specific errors.
- [ ] Displayed all validation errors.
- [ ] Preserved old form input.
- [ ] Tested invalid form submissions.
- [ ] Created a Form Request class.
- [ ] Used validated data when saving records.

---

## 17. Official Laravel Documentation

- [Validation](https://laravel.com/docs/13.x/validation)
- [Form Request Validation](https://laravel.com/docs/13.x/validation#form-request-validation)
- [Requests](https://laravel.com/docs/13.x/requests)
- [Blade Templates](https://laravel.com/docs/13.x/blade)

---

## Next Lesson: Lesson 10 – Authentication and Authorization

In the next lesson, we will learn:

- What authentication and authorization mean.
- How Laravel authentication works.
- User registration and login.
- Protecting routes with authentication middleware.
- Restricting access to application features.
- Understanding user permissions.

We will prepare the Task Management System for multiple users.