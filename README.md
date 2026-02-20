# claude-laravel

Claude Code plugin providing Laravel and PHP skill files for modern Laravel development.

## What's Included

| Skill | When to Use |
|-------|-------------|
| **developing-with-prism** | Working with Prism PHP package for LLM integration (text generation, structured output, embeddings, image generation, audio processing, streaming, tools/function calling). Supports OpenAI, Anthropic, Gemini, Mistral, Groq, XAI, DeepSeek, OpenRouter, Ollama, VoyageAI, ElevenLabs. |
| **laravel-api** | Building API endpoints, working with API resources, Sanctum authentication, route structure, and error responses. |
| **laravel-dtos** | Creating or modifying Data Transfer Objects using Spatie Laravel Data. Handling API request/response transformation, passing structured data between layers, working with model JSON casts. |
| **laravel-tall** | TALL stack development (Livewire, Alpine.js, Tailwind CSS). Building reactive components, interactive UIs, and utility-first styling. |
| **livewire-datatables** | Rappasoft Laravel Livewire Tables for tabular data display. Creating list/index pages with sorting, filtering, and searching capabilities. |
| **php-standards** | PHP 8.3 coding standards, PSR-12, strict types, naming conventions, docblock rules, and service/repository patterns. |
| **sail-workflow** | Laravel Sail development workflow commands for starting the stack, running tests, linting, and common Artisan tasks. |

## Installation

```bash
# Add the plugin marketplace (if not already added)
/plugin marketplace add darkmodesyndicate/claude-laravel

# Install the plugin
/plugin install claude-laravel
```

## Usage

Skills are invoked with the `/laravel:` namespace:

```
/laravel:laravel-tall
/laravel:laravel-api
/laravel:developing-with-prism
/laravel:laravel-dtos
/laravel:livewire-datatables
/laravel:php-standards
/laravel:sail-workflow
```

### Example Workflow

Starting a new TALL stack feature:
```
/laravel:laravel-tall
/laravel:livewire-datatables
```

Building an API endpoint:
```
/laravel:laravel-api
/laravel:laravel-dtos
/laravel:php-standards
```

Working with AI/LLM integration:
```
/laravel:developing-with-prism
/laravel:php-standards
```

Running your development environment:
```
/laravel:sail-workflow
```

## Skill Deep Dive

### developing-with-prism

Complete guide for integrating LLMs into your Laravel application using Prism:
- Text generation and structured output
- Embeddings for semantic search
- Image and audio generation
- Real-time streaming responses
- Tools and function calling
- Support for 15+ LLM providers

**Invoke when:** Adding AI features, working with LLM APIs, or building AI-powered Laravel applications

### laravel-api

Laravel API development conventions:
- RESTful route organization
- API resource transformers
- Sanctum token authentication
- Standardized error responses
- Request validation patterns

**Invoke when:** Creating API endpoints, building API controllers, or working with API consumers

### laravel-dtos

Spatie Laravel Data patterns:
- Creating Data objects for type safety
- Request/response transformation
- Model JSON casting
- Validation rules in DTOs
- Data collection handling

**Invoke when:** Passing structured data between layers, handling API transformations, or ensuring type safety

### laravel-tall

TALL stack (Tailwind, Alpine.js, Livewire, Laravel) conventions:
- Livewire component architecture
- Alpine.js reactive patterns
- Tailwind CSS utility patterns
- Component communication
- Form handling and validation

**Invoke when:** Building reactive UIs, creating Livewire components, or working with frontend interactivity

### livewire-datatables

Rappasoft Laravel Livewire Tables patterns:
- Table component setup
- Column definitions and formatting
- Sorting and filtering
- Pagination configuration
- Bulk actions and row actions

**Invoke when:** Creating data tables, list pages, or admin interfaces with tabular data

### php-standards

PHP 8.3 coding standards:
- Strict types declaration (required)
- Type hints and return types
- PSR-12 formatting
- Docblock requirements
- Naming conventions
- Service/repository patterns

**Invoke when:** Writing PHP code, conducting code reviews, or refactoring

### sail-workflow

Laravel Sail development commands:
- Starting/stopping containers
- Running tests (PHPUnit, Pest, Dusk)
- Artisan commands
- Database migrations
- Code linting and formatting
- Queue and scheduler management

**Invoke when:** Starting a development session, running tests, or executing common Laravel tasks

## Pairing with claude-workflow

Best used alongside [darkmodesyndicate/claude-workflow](https://github.com/darkmodesyndicate/claude-workflow) for the complete development loop:

1. **`/workflow:session-start`** - Load project context
2. **`/laravel:laravel-tall`** (or relevant skill) - Load framework conventions
3. **`/workflow:plan`** - Plan your feature
4. **`/workflow:implement`** - Build it
5. **`/workflow:review`** - Check for issues
6. **`/workflow:test`** - Add tests
7. **`/workflow:commit`** - Commit with conventional format
8. **`/workflow:session-end`** - Save session knowledge

## Philosophy

**Framework-specific, convention-driven:**
- Skills provide precise Laravel and PHP conventions
- Patterns match modern Laravel best practices
- Type safety and strict standards enforced
- Real-world examples from production codebases

**Tool-specific syntax, universal conventions:**
- Skills use Claude Code features (subagent references, Task tool)
- Underlying Laravel/PHP patterns work in any editor
- Project `AI_CONTEXT.md` files are tool-agnostic

## Requirements

- Laravel 11.x or higher (most patterns apply to Laravel 10.x)
- PHP 8.3+ (for strict types and modern features)
- Composer for dependency management

## License

MIT

## Contributing

Issues and pull requests welcome at [github.com/darkmodesyndicate/claude-laravel](https://github.com/darkmodesyndicate/claude-laravel)
