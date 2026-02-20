---
name: livewire-datatables
description: Rappasoft Laravel Livewire Tables conventions for tabular data display. Invoke when creating list/index pages with sorting, filtering, or searching capabilities.
---

# Livewire DataTables (Rappasoft Laravel Livewire Tables)

**IMPORTANT**: For all tabular data display, use Laravel Livewire Tables unless there's a specific reason to avoid it.

## When to Use DataTables

✅ **Use for:**
- Any list/index page with multiple rows of data
- Data that needs sorting, filtering, or searching
- Tables that might grow large (handles pagination automatically)
- Consistent UX across the application

❌ **Do NOT use for:**
- Simple, static data display (< 5 rows, no need for sorting/filtering)
- Non-tabular layouts (cards, grids, etc.)
- Highly custom interactive tables with complex client-side state

## Basic Pattern

```php
// app/Livewire/EntityName/Table.php
use Rappasoft\LaravelLivewireTables\DataTableComponent;
use Rappasoft\LaravelLivewireTables\Views\Column;
use Rappasoft\LaravelLivewireTables\Views\Filters\SelectFilter;

class Table extends DataTableComponent
{
    public function configure(): void
    {
        $this->setPrimaryKey('id')
            ->setDefaultSort('created_at', 'desc')
            ->setPerPageAccepted([15, 25, 50, 100])
            ->setPerPage(20)
            ->setFiltersEnabled()
            ->setFiltersVisibilityEnabled()
            ->setSearchEnabled();
    }

    public function columns(): array
    {
        return [
            Column::make('Name', 'name')
                ->sortable()
                ->searchable(),

            Column::make('Status', 'status')
                ->sortable()
                ->format(fn ($value, $row) => \view('components.table.entity-status', [
                    'status' => $value,
                ])->render())
                ->html(),

            Column::make('Actions', 'id')
                ->format(fn ($value, $row) => \view('components.table.entity-actions', [
                    'entity' => $row,
                ])->render())
                ->html()
                ->excludeFromColumnSelect(),
        ];
    }

    public function builder(): Builder
    {
        // By default, DataTables only loads columns specified in columns() (more performant)
        // Use ->select('*') when you need additional columns not in columns()
        // (e.g., for conditional rendering in buttons/badges)
        return EntityModel::query()->select('*');
    }

    public function filters(): array
    {
        return [
            SelectFilter::make('Status')
                ->options(['' => 'All', 'active' => 'Active', 'inactive' => 'Inactive'])
                ->filter(fn (Builder $builder, string $value) => $value ? $builder->where('status', $value) : $builder),
        ];
    }
}
```

## Index Component Pattern

```php
// app/Livewire/EntityName/Index.php - Keep it simple, just render the table
class Index extends Component
{
    public function render(): View
    {
        return \view('livewire.entity-name.index')->layout('components.admin-layout');
    }
}
```

## View Pattern

```blade
{{-- resources/views/livewire/entity-name/index.blade.php --}}
<div>
    <x-slot name="header">Entity Name</x-slot>

    <div class="space-y-6">
        {{-- Flash Messages --}}
        @if(session()->has('message'))
            <div class="p-4 bg-green-100 border border-green-400 text-green-700 rounded-lg">
                {{ session('message') }}
            </div>
        @endif

        {{-- Table --}}
        <div class="p-4 bg-white border border-gray-300 rounded-lg shadow-sm">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">All Records</h2>
            <livewire:entity-name.table />
        </div>
    </div>
</div>
```

## Best Practices

✅ **Use Blade views for column formatting** - Create view components in `resources/views/components/table/` instead of inline HTML

✅ **Use `->select('*')` when needed** - Only when you need columns beyond those in columns() array (e.g., for conditional logic in views). Otherwise, let DataTables optimize by loading only specified columns

✅ **Create reusable view components** - Each formatted column should have its own blade component

❌ **Never use heredoc/inline HTML** - Harder to maintain, more error-prone, no syntax highlighting

## Benefits

- Built-in sorting, filtering, searching, pagination
- Consistent UX across all tables in the application
- Significantly less code to maintain
- Easy to add columns or filters later
- Better performance for large datasets
- Clean separation of concerns with view components

## Package Documentation

https://rappasoft.com/docs/laravel-livewire-tables