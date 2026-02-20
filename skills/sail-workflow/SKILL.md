---
name: sail-workflow
description: Laravel Sail development workflow commands for starting the stack, running tests, linting, and common Artisan tasks. Invoke at the start of any development session using Laravel Sail.
---

# Laravel Sail Workflow

## Starting the Stack

```bash
# Start all containers in the background
sail up -d

# Start Vite dev server for frontend assets (run in a separate terminal)
npm run dev

# Stop all containers
sail down
```

## Running Tests

```bash
# Run the full test suite
sail artisan test

# Run a specific test class
sail artisan test --filter=ClassName

# Run a specific test method
sail artisan test --filter=testMethodName

# Run tests in a specific file
sail artisan test tests/Feature/UserTest.php

# Run with coverage (if Xdebug is configured)
sail artisan test --coverage
```

Always use `sail artisan test` - do not invoke `./vendor/bin/phpunit` directly.

## Code Style

```bash
# Check style violations (PHP_CodeSniffer)
sail composer check-style

# Auto-fix style violations (phpcbf)
sail composer fix-style

# Fix a specific file
./vendor/bin/phpcbf path/to/File.php
```

Run `fix-style` after every PHP file edit. If only fixing a specific file, use `./vendor/bin/phpcbf` directly.

## Code Quality Tools

### GrumPHP (if configured)

```bash
# Runs automatically on git commit via pre-commit hook
# To run manually:
sail composer exec grumphp run

# Skip GrumPHP on commit (not recommended)
git commit --no-verify
```

**Auto-fix Workflow:**
When GrumPHP fails on commit:
1. Check output for fixable errors marked with `[x]`
2. Run `./vendor/bin/phpcbf <files>` to auto-fix spacing, FQN, etc.
3. Manually fix remaining errors (return types, strict types, unused params)
4. Re-run commit

### Laravel Pint

```bash
# Format code using Laravel Pint
sail artisan pint

# Format specific files
sail artisan pint path/to/File.php
```

## Migration Safety

> **CRITICAL:** Check your project's `AI.md` for project-specific migration restrictions. Some projects forbid destructive commands like `migrate:fresh`, `migrate:refresh`, or `db:wipe`.

**Safe Commands (All Projects):**
```bash
sail artisan migrate
sail artisan migrate:rollback
sail artisan migrate:status
```

**Dangerous Commands (Check AI.md First):**
```bash
sail artisan migrate:fresh    # DESTROYS ALL DATA
sail artisan migrate:refresh  # DESTROYS ALL DATA
sail artisan db:wipe          # DESTROYS ALL DATA
```

**NEVER run destructive migration commands without explicit user confirmation.**

## Cache Clearing

```bash
# Clear all caches at once
sail artisan cache:clear && sail artisan config:clear && sail artisan route:clear && sail artisan view:clear
```

## Asset Compilation

```bash
# Development build (with source maps)
npm run dev

# Production build (minified, hashed filenames)
npm run build
```

## Common Artisan Commands

```bash
# Generate a new model with migration
sail artisan make:model ModelName -m

# Generate a controller
sail artisan make:controller ControllerName

# Generate a Livewire component
sail artisan make:livewire ComponentName

# Generate a migration
sail artisan make:migration create_table_name_table

# Generate a seeder
sail artisan make:seeder SeederClassName

# Generate a job
sail artisan make:job JobName

# Generate a notification
sail artisan make:notification NotificationName

# Generate a policy
sail artisan make:policy PolicyName --model=ModelName

# Generate a Form Request
sail artisan make:request RequestName

# View all registered routes
sail artisan route:list

# Tinker (interactive REPL)
sail artisan tinker
```

Always use `sail artisan make:*` commands to generate files rather than creating them manually.

## Queue Management

```bash
# Run queue worker
sail artisan queue:work

# Run queue worker for a specific queue
sail artisan queue:work --queue=high,default

# Restart queue workers (after deploying code changes)
sail artisan queue:restart
```

## Database

```bash
# Run pending migrations
sail artisan migrate

# Run a specific seeder
sail artisan db:seed --class=ClassName

# Run all seeders
sail artisan db:seed
```

> **Note:** Project-specific restrictions on which migration commands are safe to run are defined in each project's `AI.md`. Check there before running destructive operations.

## Troubleshooting

```bash
# Rebuild containers (after composer.json or Dockerfile changes)
sail build --no-cache

# Check container logs
sail logs

# Access the application container shell
sail shell

# Check which containers are running
sail ps
```
