---
name: laravel-api
description: Laravel API conventions for Data objects, Sanctum auth, route structure, and error responses. Invoke at the start of any session involving API endpoints, controllers, or API consumers.
---

# Laravel API Conventions

## Pattern Selection

**Check for `spatie/laravel-data` installation first:**
- If installed → use **Data objects** for both requests and responses (primary pattern below)
- If not installed → fall back to Resources/FormRequests (see reference section at bottom)

To check: look for `spatie/laravel-data` in `composer.json` dependencies, or check for `app/Data/` directory.

---

## Data Objects (Primary Pattern)

**When `spatie/laravel-data` is installed, use Data objects for all API requests and responses.**

See the `laravel-dtos` skill for comprehensive DTO patterns, formatters, transformers, and testing.

### Request Validation with Data Objects

Data objects handle validation, type casting, and transformation in a single class:

```php
<?php

declare(strict_types=1);

namespace App\Data;

use Spatie\LaravelData\Attributes\Validation\Email;
use Spatie\LaravelData\Attributes\Validation\Max;
use Spatie\LaravelData\Attributes\Validation\Required;
use Spatie\LaravelData\Attributes\Validation\Unique;
use Spatie\LaravelData\Data;

class CreateUserData extends Data
{
    public function __construct(
        #[Required, Max(255)]
        public string $name,

        #[Required, Email, Unique('users', 'email')]
        public string $email,

        #[Required, Password, Confirmed]
        public string $password,
    ) {
        $this->email = strtolower(trim($this->email));
    }
}
```

**Alternative validation approach:** You can still use FormRequests for validation, then transform into Data objects:

```php
class CreateUserRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'name'  => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'unique:users'],
            'password' => ['required', 'confirmed', 'min:8'],
        ];
    }
}

// In controller
$data = CreateUserData::from($request->validated());
```

### Response Formatting with Data Objects

Return Data objects directly for API responses - they automatically serialize to JSON:

```php
use Spatie\LaravelData\PaginatedDataCollection;

// Single resource
return UserData::from($user);

// Collection
return UserData::collect($users);

// With HTTP status
return response()->json(UserData::from($user), 201);

// Paginated collection (use PaginatedDataCollection)
return UserData::collect(User::paginate(15), PaginatedDataCollection::class);
```

### Response Data Class Pattern

```php
<?php

declare(strict_types=1);

namespace App\Data;

use Carbon\CarbonImmutable;
use Spatie\LaravelData\Data;

class UserData extends Data
{
    public function __construct(
        public int $id,
        public string $name,
        public string $email,
        public CarbonImmutable $createdAt,
    ) {}
}
```

Data objects automatically:
- Cast dates to ISO 8601 format
- Handle nested Data objects
- Support collections via `::collect()`
- Transform to arrays/JSON via `toArray()` and `toJson()`

### Pagination Format

Use Spatie's built-in `PaginatedDataCollection` for paginated responses:

```php
use Spatie\LaravelData\PaginatedDataCollection;

// Automatically wraps Laravel paginator with data, links, and meta
return UserData::collect(User::paginate(15), PaginatedDataCollection::class);
```

This automatically generates the standard Laravel pagination format:

```json
{
  "data": [...],
  "links": { "first": "...", "last": "...", "prev": null, "next": "..." },
  "meta": { "current_page": 1, "last_page": 5, "per_page": 15, "total": 73 }
}
```

**Do not manually wrap pagination data** - the `PaginatedDataCollection` handles it automatically.

---

## Controllers (Data Objects Pattern)

### Thin Controllers, Service Layer

- Controllers handle HTTP request/response only
- Business logic lives in Service classes
- Controllers receive Data objects, pass to services, return Data objects
- Keep to standard CRUD methods: `index`, `store`, `show`, `update`, `destroy`
- Create a new controller for non-CRUD actions rather than adding custom methods

### Using Data validation attributes:

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers;

use App\Data\CreateUserData;
use App\Data\UserData;
use App\Services\UserService;
use Spatie\LaravelData\PaginatedDataCollection;

class UsersController extends Controller
{
    public function index()
    {
        return UserData::collect(User::paginate(15), PaginatedDataCollection::class);
    }

    public function store(CreateUserData $data, UserService $userService)
    {
        $user = $userService->create($data);

        return response()->json(UserData::from($user), 201);
    }
}
```

### Using FormRequests + Data transformation:

```php
use Spatie\LaravelData\PaginatedDataCollection;

class UsersController extends Controller
{
    public function index()
    {
        return UserData::collect(User::paginate(15), PaginatedDataCollection::class);
    }

    public function store(
        CreateUserRequest $request,
        UserService $userService,
    ) {
        $data = CreateUserData::from($request->validated());

        $user = $userService->create($data);

        return response()->json(UserData::from($user), 201);
    }
}
```

Both approaches are valid - choose based on:
- Data attributes: Better for simple validation, keeps everything in one class
- FormRequests: Better for complex authorization logic or custom validation messages

---

## Authentication

### Sanctum

- API authentication uses Laravel Sanctum
- Token-based auth for mobile/SPA clients: `Authorization: Bearer {token}`
- SPA cookie-based auth for same-domain frontends: session cookies via `/sanctum/csrf-cookie`
- Protect routes with `auth:sanctum` middleware

### Token Abilities

```php
$token = $user->createToken('token-name', ['read:orders', 'write:orders']);
$token->tokenable->tokenCan('read:orders');
```

---

## Route Conventions

### Naming and Structure

```php
// Resource routes (preferred)
Route::apiResource('users', UsersController::class);

// Manual routes
Route::get('/users', [UsersController::class, 'index'])->name('api.users.index');
Route::post('/users', [UsersController::class, 'store'])->name('api.users.store');
```

- URLs: plural, kebab-case (`/error-occurrences`, `/job-applications`)
- Route names: camelCase (`api.jobApplications.index`)
- Route parameters: camelCase (`{userId}`, `{jobApplicationId}`)
- Group API routes under `routes/api.php`
- Prefix with version where versioning is in use: `/v1/users`

### Nesting

Limit to one level of nesting for simplicity:

```
GET /jobs/{job}/applications      ✓
GET /jobs/{job}/applications/{application}/notes  ✗ (too deep)
GET /application-notes/{note}     ✓ (flatten instead)
```

---

## Error Responses

### HTTP Status Codes

- `200` - Successful GET, PUT, PATCH
- `201` - Successful POST (resource created)
- `204` - Successful DELETE (no content)
- `400` - Bad request (malformed input)
- `401` - Unauthenticated
- `403` - Unauthorized (authenticated but forbidden)
- `404` - Resource not found
- `422` - Validation failed (Laravel default for failed validation)
- `500` - Server error

### Validation Error Format (Laravel default)

```json
{
  "message": "The email field is required.",
  "errors": {
    "email": ["The email field is required."]
  }
}
```

Both Data validation attributes and FormRequests return this format automatically.

### Custom Error Responses

```php
return response()->json(['message' => 'Resource not found.'], 404);
```

---

## Versioning

- Version in URL path when breaking changes are possible: `/api/v1/`, `/api/v2/`
- Group versioned routes in separate route files: `routes/api_v1.php`
- New versions inherit previous version routes where unchanged
- Use transformers for version-specific Data object mappings (see `laravel-dtos` skill)

---

## Resources & FormRequests (Reference/Fallback)

**Use this pattern when:**
- `spatie/laravel-data` is not installed
- Working with legacy codebases using Resources
- Simple APIs where Data objects are overkill

### Resource Classes

Always use Laravel API Resources for API responses - never return Eloquent models or raw arrays directly.

```php
// Single resource
return new UserResource($user);

// Collection
return UserResource::collection($users);

// With pagination (auto-detected from paginator)
return UserResource::collection(User::paginate(15));
```

### Resource Class Pattern

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'         => $this->id,
            'name'       => $this->name,
            'email'      => $this->email,
            'created_at' => $this->created_at->toISOString(),
        ];
    }
}
```

### Pagination Format

Laravel's built-in pagination via `->paginate()` returns:

```json
{
  "data": [...],
  "links": { "first": "...", "last": "...", "prev": null, "next": "..." },
  "meta": { "current_page": 1, "last_page": 5, "per_page": 15, "total": 73 }
}
```

Do not reshape this structure - use it as-is.

### Form Requests for Validation

Always use Form Request classes for API validation - never inline `$request->validate()`:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreUserRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'name'  => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'unique:users'],
        ];
    }
}
```

### Controller Pattern with Resources

```php
class UsersController extends Controller
{
    public function store(StoreUserRequest $request, UserService $userService): JsonResponse
    {
        $user = $userService->create($request->validated());

        return response()->json(new UserResource($user), 201);
    }
}
```
