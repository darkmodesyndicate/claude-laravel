---
name: laravel-tall
description: TALL stack conventions for Livewire, Alpine.js, and Tailwind CSS in Laravel projects. Invoke at the start of any session involving Livewire components, Alpine.js interactivity, or Tailwind styling.
---

# TALL Stack Conventions

## Livewire Components

### File Structure
- Component classes: `app/Livewire/`
- Blade views: `resources/views/livewire/`
- Full-page components are registered as routes in `routes/web.php`
- Embedded components are used directly in Blade templates via `<livewire:component-name />`

### Component Class Rules
- All `render()` methods must return `\Illuminate\Contracts\View\View`
- Use `use Illuminate\Contracts\View\View;` import, then declare `: View` return type
- Use `\view('livewire.component-name')` with backslash prefix on global function
- Lifecycle hooks: `mount()`, `updated()`, `updatedFoo()`, `hydrate()`, `dehydrate()`
- Use `#[Computed]` attribute for computed properties (Livewire v3)
- Use `#[Lazy]` for lazy-loaded components
- Use `#[Validate]` or `$rules` property for validation; call `$this->validate()` in submit methods
- Use Form Objects (`Livewire\Form`) for complex forms with their own validation

### Real-time Validation
```php
#[Validate('required|email')]
public string $email = '';
```

### Events
- Dispatch from component: `$this->dispatch('event-name', payload: $data)`
- Listen in component: `#[On('event-name')]` attribute on method
- Dispatch to browser: `$this->dispatch('event-name')->to('component-name')`
- Event names: camelCase, domain-prefixed (e.g., `userUpdated`, `invoicePaid`)

### Wire Directives
- `wire:model` for two-way binding (add `.live` for real-time, `.blur` for on-blur)
- `wire:click`, `wire:submit.prevent` for event handling
- `wire:navigate` for SPA-style navigation (replaces `<a href>` in Livewire SPA mode)
- `wire:loading` / `wire:loading.remove` for loading states
- `wire:key` for list items to maintain identity

### Polling & Lazy Loading
```blade
<livewire:stats-widget lazy />
<div wire:poll.5s>...</div>
```

---

## Alpine.js

### Core Directives
- `x-data="{ ... }"` - declares component state (keep minimal, close to the HTML)
- `x-show` / `x-if` - conditional visibility (`x-show` keeps in DOM, `x-if` removes it)
- `x-bind:attr` or `:attr` - dynamic attribute binding
- `x-on:event` or `@event` - event listeners
- `x-model` - two-way binding for form inputs
- `x-text` / `x-html` - text/HTML content binding
- `x-ref` - DOM element references
- `x-transition` - built-in transition helpers

### Livewire + Alpine Integration
- Access Livewire component from Alpine: `$wire.propertyName`, `$wire.methodName()`
- Listen to Livewire events: `window.addEventListener('event-name', e => { ... })`
- Dispatch to Livewire from Alpine: `$dispatch('event-name', { data })` or `$wire.dispatch('event-name')`

### Patterns
- Keep Alpine state inline in the template, not in separate JS files
- Use `x-data` on the smallest container that needs the state
- For shared state, use Alpine stores: `Alpine.store('name', { ... })`
- Prefer `x-show` over `x-if` for toggled UI elements (avoids DOM re-creation)

---

## Tailwind CSS

### Class Organization (within a single element)
Layout → Box model → Typography → Visual → Interactive
```blade
<div class="flex items-center gap-4 px-6 py-3 text-sm font-medium text-gray-700 bg-white rounded-lg shadow hover:bg-gray-50">
```

### Responsive Design
- Mobile-first: base classes apply to mobile, add `sm:`, `md:`, `lg:`, `xl:` for breakpoints
- Common breakpoints: `sm` (640px), `md` (768px), `lg` (1024px), `xl` (1280px)

### Component Patterns
- Use `@apply` in CSS files only for truly reusable base component styles
- Prefer utility classes in templates over `@apply` for one-off styling
- Use Tailwind config (`tailwind.config.js`) for project colour/font tokens

### Dark Mode
- Use `dark:` variant when dark mode is enabled in config
- Config: `darkMode: 'class'` (toggle by adding `dark` class to `<html>`)

---

## Full-Page Livewire Components

Register as routes instead of using controllers:
```php
// routes/web.php
Route::get('/users', UsersList::class)->name('users.index');
```

Layout: use `#[Layout('layouts.app')]` attribute on the component class, or declare:
```php
public function render(): View
{
    return \view('livewire.users-list')->layout('layouts.app');
}
```

---

## When to Use What

| Scenario | Approach |
|---|---|
| Simple toggle, dropdown | Alpine.js only (`x-data`, `x-show`) |
| Form with server validation | Livewire with `#[Validate]` |
| Page with server-rendered data | Full-page Livewire component |
| Real-time updates | Livewire polling or events |
| Complex UI interaction (no server) | Alpine.js |
| Livewire + client-side enhancement | `$wire` bridge |
