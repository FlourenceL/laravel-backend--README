# Laravel Backend - Complete Guide

A comprehensive guide to building backend APIs with Laravel, including how to use the Laravel documentation and step-by-step CRUD API implementation.

## Table of Contents

1. [How to Use Laravel Documentation](#how-to-use-laravel-documentation)
2. [Step-by-Step CRUD Backend API Guide](#step-by-step-crud-backend-api-guide)
3. [Testing Your API](#testing-your-api)
4. [Best Practices](#best-practices)
5. [Troubleshooting](#troubleshooting)
6. [Additional Resources](#additional-resources)

---

## How to Use Laravel Documentation

Laravel has excellent documentation at [laravel.com/docs](https://laravel.com/docs). Here's how to navigate and use it effectively:

### Step 1: Choose Your Laravel Version
- Visit [https://laravel.com/docs](https://laravel.com/docs)
- Select your Laravel version from the dropdown (e.g., 10.x, 11.x)
- **Important**: Always use documentation matching your installed Laravel version

### Step 2: Understand the Documentation Structure

The Laravel docs are organized into logical sections:

#### **Getting Started**
- Installation
- Configuration
- Directory Structure
- Deployment

#### **Architecture Concepts**
- Request Lifecycle
- Service Container
- Service Providers
- Facades

#### **The Basics**
- Routing
- Middleware
- Controllers
- Requests
- Responses
- Views
- Blade Templates
- Asset Bundling
- URL Generation
- Session
- Validation
- Error Handling
- Logging

#### **Digging Deeper**
- Artisan Console
- Broadcasting
- Cache
- Collections
- Events
- File Storage
- Helpers
- HTTP Client
- Localization
- Mail
- Notifications
- Package Development
- Queues
- Rate Limiting
- Task Scheduling

#### **Database**
- Getting Started
- Query Builder
- Pagination
- Migrations
- Seeding
- Redis

#### **Eloquent ORM**
- Getting Started
- Relationships
- Collections
- Mutators / Casts
- API Resources
- Serialization

#### **Security**
- Authentication
- Authorization
- Email Verification
- Encryption
- Hashing
- Password Reset

### Step 3: Search Efficiently
- Use the search bar at the top of the documentation
- Search for specific features (e.g., "eloquent relationships", "validation rules")
- Browse the sidebar for related topics

### Step 4: Follow Code Examples
- Each documentation page includes practical code examples
- Copy and adapt examples to your project
- Pay attention to namespace imports and use statements

### Step 5: Check API Documentation
- For detailed class/method information, visit [https://laravel.com/api/](https://laravel.com/api/)
- Choose your version
- Browse classes, interfaces, and traits

### Step 6: Use Artisan Help
```bash
# List all available artisan commands
php artisan list

# Get help for a specific command
php artisan help make:controller

# View all routes
php artisan route:list
```

### Step 7: Leverage Laravel's Learning Resources
- **Laracasts**: [laracasts.com](https://laracasts.com) - Video tutorials
- **Laravel News**: [laravel-news.com](https://laravel-news.com) - Latest updates
- **Laravel Bootcamp**: [bootcamp.laravel.com](https://bootcamp.laravel.com) - Guided tutorials

---

## Step-by-Step CRUD Backend API Guide

This guide will walk you through creating a complete CRUD (Create, Read, Update, Delete) API for a "Task" resource.

### Prerequisites
- PHP >= 8.1
- Composer
- MySQL/PostgreSQL/SQLite
- Postman or similar API testing tool (optional)

### Step 1: Install Laravel

```bash
# Install Laravel via Composer
composer create-project laravel/laravel task-api

# Navigate to project directory
cd task-api

# Start development server
php artisan serve
```

Your Laravel app will be available at `http://localhost:8000`

### Step 2: Configure Database

Edit `.env` file in your project root:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=task_api
DB_USERNAME=your_username
DB_PASSWORD=your_password
```

For SQLite (simpler for development):
```env
DB_CONNECTION=sqlite
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=laravel
# DB_USERNAME=root
# DB_PASSWORD=
```

Create SQLite database:
```bash
touch database/database.sqlite
```

### Step 3: Create Migration

```bash
# Create a migration for tasks table
php artisan make:migration create_tasks_table
```

Edit the migration file in `database/migrations/YYYY_MM_DD_XXXXXX_create_tasks_table.php`:

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
            $table->enum('status', ['pending', 'in_progress', 'completed'])->default('pending');
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

Run the migration:
```bash
php artisan migrate
```

### Step 4: Create Model

```bash
# Create Task model
php artisan make:model Task
```

Edit `app/Models/Task.php`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Task extends Model
{
    use HasFactory;

    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'title',
        'description',
        'status',
        'due_date',
    ];

    /**
     * The attributes that should be cast.
     *
     * @var array<string, string>
     */
    protected $casts = [
        'due_date' => 'date',
    ];
}
```

### Step 5: Create Controller

```bash
# Create API controller for Task
php artisan make:controller Api/TaskController --api
```

Edit `app/Http/Controllers/Api/TaskController.php`:

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Task;
use Illuminate\Http\Request;
use Illuminate\Http\JsonResponse;

class TaskController extends Controller
{
    /**
     * Display a listing of tasks.
     */
    public function index(): JsonResponse
    {
        // Use pagination for better performance with large datasets
        $tasks = Task::paginate(15);
        
        return response()->json([
            'success' => true,
            'data' => $tasks->items(),
            'meta' => [
                'current_page' => $tasks->currentPage(),
                'total' => $tasks->total(),
                'per_page' => $tasks->perPage(),
                'last_page' => $tasks->lastPage(),
            ]
        ], 200);
    }

    /**
     * Store a newly created task.
     */
    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'status' => 'nullable|in:pending,in_progress,completed',
            'due_date' => 'nullable|date',
        ]);

        $task = Task::create($validated);

        return response()->json([
            'success' => true,
            'message' => 'Task created successfully',
            'data' => $task
        ], 201);
    }

    /**
     * Display the specified task.
     */
    public function show(Task $task): JsonResponse
    {
        return response()->json([
            'success' => true,
            'data' => $task
        ], 200);
    }

    /**
     * Update the specified task.
     */
    public function update(Request $request, Task $task): JsonResponse
    {
        $validated = $request->validate([
            'title' => 'sometimes|required|string|max:255',
            'description' => 'nullable|string',
            'status' => 'nullable|in:pending,in_progress,completed',
            'due_date' => 'nullable|date',
        ]);

        $task->update($validated);

        return response()->json([
            'success' => true,
            'message' => 'Task updated successfully',
            'data' => $task
        ], 200);
    }

    /**
     * Remove the specified task.
     */
    public function destroy(Task $task): JsonResponse
    {
        $task->delete();

        return response()->json([
            'success' => true,
            'message' => 'Task deleted successfully'
        ], 200);
    }
}
```

### Step 6: Define API Routes

Edit `routes/api.php`:

```php
<?php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\TaskController;

/*
|--------------------------------------------------------------------------
| API Routes
|--------------------------------------------------------------------------
*/

Route::prefix('v1')->group(function () {
    Route::apiResource('tasks', TaskController::class);
});

// Optional: Get authenticated user
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});
```

### Step 7: Test Your API Endpoints

Your API endpoints are now available:

#### **GET** - List all tasks
```bash
curl http://localhost:8000/api/v1/tasks
```

#### **POST** - Create a new task
```bash
curl -X POST http://localhost:8000/api/v1/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Complete Laravel Tutorial",
    "description": "Learn how to build CRUD APIs",
    "status": "in_progress",
    "due_date": "2026-12-31"
  }'
```

#### **GET** - Get a specific task
```bash
curl http://localhost:8000/api/v1/tasks/1
```

#### **PUT/PATCH** - Update a task
```bash
curl -X PUT http://localhost:8000/api/v1/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Updated Task Title",
    "status": "completed"
  }'
```

#### **DELETE** - Delete a task
```bash
curl -X DELETE http://localhost:8000/api/v1/tasks/1
```

### Step 8: Add API Resource (Optional - For Better Response Formatting)

```bash
php artisan make:resource TaskResource
```

Edit `app/Http/Resources/TaskResource.php`:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class TaskResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'description' => $this->description,
            'status' => $this->status,
            'due_date' => $this->due_date?->format('Y-m-d'),
            'created_at' => $this->created_at->toDateTimeString(),
            'updated_at' => $this->updated_at->toDateTimeString(),
        ];
    }
}
```

Update controller to use resources:

```php
use App\Http\Resources\TaskResource;

public function index(): JsonResponse
{
    $tasks = Task::paginate(15);
    return response()->json([
        'success' => true,
        'data' => TaskResource::collection($tasks->items()),
        'meta' => [
            'current_page' => $tasks->currentPage(),
            'total' => $tasks->total(),
            'per_page' => $tasks->perPage(),
            'last_page' => $tasks->lastPage(),
        ]
    ], 200);
}

public function show(Task $task): JsonResponse
{
    return response()->json([
        'success' => true,
        'data' => new TaskResource($task)
    ], 200);
}
```

### Step 9: Add Database Seeding (Optional)

```bash
php artisan make:seeder TaskSeeder
```

Edit `database/seeders/TaskSeeder.php`:

```php
<?php

namespace Database\Seeders;

use App\Models\Task;
use Illuminate\Database\Seeder;

class TaskSeeder extends Seeder
{
    public function run(): void
    {
        Task::create([
            'title' => 'Learn Laravel Basics',
            'description' => 'Complete the Laravel fundamentals course',
            'status' => 'completed',
            'due_date' => '2026-01-15',
        ]);

        Task::create([
            'title' => 'Build CRUD API',
            'description' => 'Create a RESTful API with Laravel',
            'status' => 'in_progress',
            'due_date' => '2026-02-01',
        ]);

        Task::create([
            'title' => 'Deploy Application',
            'description' => 'Deploy the app to production',
            'status' => 'pending',
            'due_date' => '2026-03-01',
        ]);
    }
}
```

Run seeder:
```bash
php artisan db:seed --class=TaskSeeder
```

### Step 10: Add Factory (Optional - For Testing)

```bash
php artisan make:factory TaskFactory
```

Edit `database/factories/TaskFactory.php`:

```php
<?php

namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;

class TaskFactory extends Factory
{
    public function definition(): array
    {
        return [
            'title' => fake()->sentence(3),
            'description' => fake()->paragraph(),
            'status' => fake()->randomElement(['pending', 'in_progress', 'completed']),
            'due_date' => fake()->dateTimeBetween('now', '+1 month'),
        ];
    }
}
```

Use in tinker:
```bash
php artisan tinker
Task::factory()->count(10)->create();
```

---

## Testing Your API

### Using Postman

1. Download and install [Postman](https://www.postman.com/downloads/)
2. Create a new collection called "Task API"
3. Add requests for each endpoint (GET, POST, PUT, DELETE)
4. Set the base URL to `http://localhost:8000/api/v1`

### Using cURL (Command Line)

Examples are provided in Step 7 above.

### Using Laravel HTTP Tests

Create a test file:
```bash
php artisan make:test TaskApiTest
```

Edit `tests/Feature/TaskApiTest.php`:

```php
<?php

namespace Tests\Feature;

use App\Models\Task;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class TaskApiTest extends TestCase
{
    use RefreshDatabase;

    public function test_can_list_tasks(): void
    {
        Task::factory()->count(3)->create();

        $response = $this->getJson('/api/v1/tasks');

        $response->assertStatus(200)
                 ->assertJsonCount(3, 'data');
    }

    public function test_can_create_task(): void
    {
        $taskData = [
            'title' => 'Test Task',
            'description' => 'Test Description',
            'status' => 'pending',
        ];

        $response = $this->postJson('/api/v1/tasks', $taskData);

        $response->assertStatus(201)
                 ->assertJsonPath('data.title', 'Test Task');
    }

    public function test_can_show_task(): void
    {
        $task = Task::factory()->create();

        $response = $this->getJson("/api/v1/tasks/{$task->id}");

        $response->assertStatus(200)
                 ->assertJsonPath('data.id', $task->id);
    }

    public function test_can_update_task(): void
    {
        $task = Task::factory()->create();

        $response = $this->putJson("/api/v1/tasks/{$task->id}", [
            'title' => 'Updated Title',
        ]);

        $response->assertStatus(200)
                 ->assertJsonPath('data.title', 'Updated Title');
    }

    public function test_can_delete_task(): void
    {
        $task = Task::factory()->create();

        $response = $this->deleteJson("/api/v1/tasks/{$task->id}");

        $response->assertStatus(200);
        $this->assertDatabaseMissing('tasks', ['id' => $task->id]);
    }
}
```

Run tests:
```bash
php artisan test
```

---

## Best Practices

### 1. **Validation**
- Always validate incoming data
- Use Form Request classes for complex validation
- Return meaningful error messages

### 2. **API Versioning**
- Version your API (e.g., `/api/v1/tasks`)
- Maintain backward compatibility when possible

### 3. **Error Handling**
- Implement consistent error responses
- Use appropriate HTTP status codes
- Log errors for debugging

### 4. **Security**
- Implement authentication (Laravel Sanctum or Passport)
- Use CORS middleware for cross-origin requests
- Validate and sanitize all inputs
- Use rate limiting to prevent abuse

### 5. **Performance**
- Use pagination for large datasets
- Implement caching where appropriate
- Optimize database queries (use eager loading)

### 6. **Documentation**
- Document your API endpoints
- Use tools like Swagger/OpenAPI
- Provide clear examples

### 7. **Code Organization**
- Follow Laravel conventions
- Use Service classes for business logic
- Keep controllers thin

---

## Troubleshooting

### Common Issues

#### "Class not found" error
```bash
composer dump-autoload
```

#### Database connection error
- Check `.env` database credentials
- Ensure database server is running
- Verify database exists

#### Route not found
```bash
php artisan route:list
php artisan route:clear
php artisan cache:clear
```

#### CORS errors
Install and configure Laravel CORS:
```bash
# Already included in Laravel 10+
# Just configure in config/cors.php
```

#### Permission errors
```bash
chmod -R 775 storage bootstrap/cache
```

---

## Additional Resources

### Official Documentation
- [Laravel Documentation](https://laravel.com/docs)
- [Laravel API Documentation](https://laravel.com/api/)
- [Laravel News](https://laravel-news.com)

### Learning Platforms
- [Laracasts](https://laracasts.com) - Premium video tutorials
- [Laravel Bootcamp](https://bootcamp.laravel.com) - Free guided tutorials
- [Laravel Daily](https://laraveldaily.com) - Tips and tutorials

### Community
- [Laravel.io Forum](https://laravel.io/forum)
- [Laracasts Forum](https://laracasts.com/discuss)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/laravel)
- [Discord](https://discord.gg/laravel)

### Tools
- [Laravel Debugbar](https://github.com/barryvdh/laravel-debugbar) - Debugging tool
- [Laravel IDE Helper](https://github.com/barryvdh/laravel-ide-helper) - IDE autocomplete
- [Laravel Telescope](https://laravel.com/docs/telescope) - Debug assistant
- [Postman](https://www.postman.com) - API testing
- [Insomnia](https://insomnia.rest) - API testing alternative

### Package Repositories
- [Packagist](https://packagist.org) - PHP package repository
- [Spatie](https://spatie.be/open-source) - High-quality Laravel packages

---

## Contributing

Feel free to contribute to this guide by submitting issues or pull requests.

## License

This guide is open-sourced and available for educational purposes.
