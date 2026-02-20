---
name: php-standards
description: PHP 8.3 coding standards, PSR-12, strict types, naming conventions, docblock rules, and service/repository patterns. Invoke when writing or reviewing PHP code.
---

# PHP Coding Standards

## Strict Types (Required)

Every PHP file must begin with:
```php
<?php

declare(strict_types=1);
```

No exceptions. This applies to all PHP files including migrations, seeders, and console commands.

## Return Types (Required)

All methods must declare return types, including `void`:
```php
public function process(): void {}
public function getUser(): User {}
public function findById(int $id): ?User {}
```

## Parameter Types (Required)

All parameters must have type hints:
```php
public function create(string $name, int $age, ?string $email = null): User {}
```

## Global PHP Functions - Backslash Prefix (Required)

All built-in PHP functions must be prefixed with `\`:
```php
\count($array);
\array_map(fn ($x) => $x * 2, $items);
\str_replace('foo', 'bar', $string);
\now();
\auth();
\view('template');
\route('name');
\round(3.5, 2);
\in_array($value, $array, true);
```

This is enforced by PHP_CodeSniffer. Do not omit the backslash.

## Nullable Types

Use short nullable syntax:
```php
?string $name     // correct
string|null $name // incorrect
```

## Class Structure - Method Order (Critical)

Methods must appear in this exact order. Never return to a higher visibility group after a lower one:

1. Constants
2. Properties (public → protected → private)
3. Magic methods (`__construct`, `__destruct`, `__toString`, etc.)
4. Public methods
5. Protected methods
6. Private methods
7. Private static methods

## Constructor Property Promotion

When all constructor parameters become properties, use promotion:
```php
public function __construct(
    private readonly UserRepository $userRepository,
    private readonly MailService $mailService,
) {}
```

## Naming Conventions

| Type | Convention | Example |
|---|---|---|
| Classes | PascalCase | `UserController`, `OrderStatus` |
| Methods | camelCase | `getUserName()`, `findByEmail()` |
| Variables | camelCase | `$firstName`, `$orderItems` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Enums | PascalCase values | `Status::Active` |
| Config files | kebab-case | `pdf-generator.php` |
| Config keys | snake_case | `chrome_path` |
| Artisan commands | kebab-case | `delete-old-records` |

## Docblock Rules

Use typed properties over docblocks. Only use docblocks when:
1. The property is inherited from a parent class that lacks type hints
2. The method returns a generic collection type that needs specificity
3. A description is genuinely needed

**For inherited properties (e.g. Eloquent `$fillable`, `$casts`, Livewire `$listeners`):**
```php
/**
 * @var array<string>
 *
 * @phpcsSuppress SlevomatCodingStandard.TypeHints.PropertyTypeHint.MissingNativeTypeHint
 * @phpcsSuppress SlevomatCodingStandard.TypeHints.PropertyTypeHint.MissingTraversableTypeHintSpecification
 */
protected $fillable = ['name', 'email'];
```

**For typed collections:**
```php
/** @return Collection<int, User> */
public function getActiveUsers(): Collection {}
```

**Never use fully qualified class names in docblocks** - import with `use` and reference the short name:
```php
use App\Models\User;
/** @return Collection<int, User> */ // correct
/** @return Collection<int, \App\Models\User> */ // incorrect
```

**Array shapes:**
```php
/** @return array{
 *     first: SomeClass,
 *     second: SomeClass,
 * }
 */
```

## Control Flow

### Happy Path Last
```php
// Guard clauses first, success case last
if (! $user) {
    return null;
}

if (! $user->isActive()) {
    throw new InactiveUserException();
}

return $this->processUser($user);
```

### No `else` - Use Early Returns
```php
// Bad
if ($condition) {
    return $a;
} else {
    return $b;
}

// Good
if ($condition) {
    return $a;
}

return $b;
```

### Always Use Curly Brackets
```php
if ($condition) {
    doSomething();  // even for single statements
}
```

### Arrow Functions and Match - Space After Keyword
```php
fn ($value) => $value * 2    // correct (space after fn)
fn($value) => $value * 2     // incorrect

match ($status) {            // correct (space after match)
    Status::Active => true,
    default => false,
}
```

## Service / Repository Pattern

### Service Classes
- Contain business logic
- Injected into controllers or other services
- Single responsibility: one domain concern per service

### Repository Classes
- Handle database queries only
- Return Eloquent models or collections
- No business logic

### Injection Rules
- Service used in **one** controller method → inject into that method directly
- Service used in **multiple** controller methods → inject via constructor

```php
// Single method use
public function store(StoreRequest $request, UserService $userService): Response
{
    return response()->json($userService->create($request->validated()));
}

// Multiple method use
public function __construct(
    private readonly UserService $userService,
) {}
```

## Exception Handling

- Use try-catch for expected exceptions
- Create custom exception classes for domain errors
- Use Laravel's exception handling for unexpected errors
- Log unexpected exceptions with context

```php
try {
    $result = $this->service->process($data);
} catch (ValidationException $e) {
    return response()->json(['error' => $e->getMessage()], 422);
}
```

## Testing

### Naming Convention
Use `testCamelCase` - NOT `test_snake_case`:
```php
public function testUserCanLogin(): void {}       // correct
public function test_user_can_login(): void {}    // incorrect
```

### Test Structure
- Feature tests: `tests/Feature/` - test full HTTP requests and database
- Unit tests: `tests/Unit/` - test individual classes/methods in isolation
- Follow Arrange-Act-Assert pattern
- Mock external services; test your own code

### Running Tests
```bash
sail artisan test
sail artisan test --filter=TestClassName
```
