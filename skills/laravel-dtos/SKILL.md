---
name: laravel-dtos
description: Data Transfer Objects using Spatie Laravel Data. Invoke when creating or modifying DTOs, passing structured data between layers, handling API request/response transformation, or working with model JSON casts.
---

# Laravel DTOs

Uses [Spatie Laravel Data](https://spatie.be/docs/laravel-data). All DTOs extend `Spatie\LaravelData\Data`.

**Rule:** Never pass multiple primitive values between layers. Wrap structured data in a DTO.

---

## Naming Conventions

| Type | Pattern | Examples |
|---|---|---|
| Response / read DTOs | `{Entity}Data` | `UserData`, `OrderData` |
| Request / write DTOs | `{Action}{Entity}Data` | `CreateUserData`, `UpdateOrderData` |
| Nested DTOs | `{Descriptor}{Entity}Data` | `ShippingAddressData`, `BillingAddressData` |

All DTOs live in `app/Data/`.

---

## Basic Structure

```php
<?php

declare(strict_types=1);

namespace App\Data;

use App\Enums\OrderStatus;
use Carbon\CarbonImmutable;
use Illuminate\Support\Collection;
use Spatie\LaravelData\Data;

class CreateOrderData extends Data
{
    public function __construct(
        public string $customerEmail,
        public ?string $notes,
        public ?CarbonImmutable $deliveryDate,
        public OrderStatus $status,
        /** @var Collection<int, OrderItemData> */
        public Collection $items,
        public ShippingAddressData $shippingAddress,
    ) {
        $this->customerEmail = \strtolower(\trim($this->customerEmail));
    }
}
```

- Always use constructor property promotion
- Always declare `declare(strict_types=1)`
- Type everything — no untyped properties
- Nullable uses `?Type`, never `Type|null`
- Collections require a `@var` PHPDoc with key and value types
- Apply formatting/normalisation inside the constructor

---

## Creating DTOs — `::from()` vs `new`

**Prefer `::from()` with an array** — the package handles casting automatically based on property types (strings → enums, strings → CarbonImmutable, arrays → nested DTOs).

```php
// Preferred — package casts automatically
$data = CreateOrderData::from([
    'customerEmail' => $request->input('customer_email'),
    'deliveryDate'  => $request->input('delivery_date'),  // string → CarbonImmutable
    'status'        => $request->input('status'),          // string → OrderStatus enum
    'items'         => $request->input('items'),           // array → Collection<OrderItemData>
    'shippingAddress' => $request->input('shipping'),
]);

// Use `new` only when values are already the correct type
// (e.g. inside test factories, or when pulling from a typed source)
$data = new CreateOrderData(
    customerEmail: $user->email,
    notes: null,
    deliveryDate: \now()->addDays(3),
    status: OrderStatus::Pending,
    items: \collect([]),
    shippingAddress: $addressData,
);
```

**Do not use `#[MapInputName]` or case mapper attributes.** Map field names explicitly at the call site — it is easier to trace and survives API version changes cleanly.

---

## Date Casting

The package casts date strings to `Carbon` or `CarbonImmutable` automatically based on the property type. Configure the expected format in `config/data.php`:

```php
// config/data.php
return [
    'date_format' => 'Y-m-d H:i:s',
];
```

Declare the property type and pass the raw string — no manual parsing needed:

```php
public ?CarbonImmutable $publishedAt,  // null or CarbonImmutable, cast from string automatically
```

---

## Nested DTOs and Collections

```php
class OrderData extends Data
{
    public function __construct(
        public CustomerData $customer,
        public ShippingAddressData $shippingAddress,
        /** @var Collection<int, OrderItemData> */
        public Collection $items,
        /** @var int[] */
        public array $tagIds,
    ) {}
}
```

Pass nested data as arrays — the package resolves them into the correct types:

```php
OrderData::from([
    'customer'        => ['name' => 'Jane', 'email' => 'jane@example.com'],
    'shippingAddress' => ['line1' => '123 Main St', 'city' => 'Auckland'],
    'items'           => [['productId' => 1, 'quantity' => 2]],
    'tagIds'          => [4, 7, 12],
]);
```

---

## Formatters

Apply data normalisation in the constructor. Keep each formatter as a focused static class in `app/Data/Formatters/`:

```php
// app/Data/Formatters/EmailFormatter.php
class EmailFormatter
{
    public static function format(string $email): string
    {
        return \strtolower(\trim($email));
    }
}
```

```php
public function __construct(
    public string $email,
    public ?string $phone,
) {
    $this->email = EmailFormatter::format($this->email);
    $this->phone = $this->phone ? PhoneFormatter::format($this->phone) : null;
}
```

---

## Static Factory Methods

For straightforward, single-source mappings, add static factory methods directly on the DTO rather than creating a separate transformer class:

```php
class CreateUserData extends Data
{
    public function __construct(
        public string $name,
        public string $email,
        public ?string $phone,
    ) {
        $this->email = EmailFormatter::format($this->email);
    }

    public static function fromRequest(CreateUserRequest $request): self
    {
        return self::from([
            'name'  => $request->input('name'),
            'email' => $request->input('email'),
            'phone' => $request->input('phone'),
        ]);
    }

    public static function fromModel(User $user): self
    {
        return self::from([
            'name'  => $user->name,
            'email' => $user->email,
            'phone' => $user->phone,
        ]);
    }
}
```

---

## Transformers (complex / multi-source mappings)

When multiple external sources map to the same DTO, or when field mapping is complex enough to warrant isolated testing, extract a dedicated transformer class:

```php
// app/Data/Transformers/Api/V1/OrderDataTransformer.php
class OrderDataTransformer
{
    public static function fromRequest(CreateOrderRequest $request): CreateOrderData
    {
        return CreateOrderData::from([
            'customerEmail'   => $request->input('customer_email'),
            'notes'           => $request->input('notes'),
            'status'          => $request->input('status'),
            'items'           => $request->input('line_items'),
            'shippingAddress' => $request->input('shipping_address'),
        ]);
    }

    public static function fromWebhook(array $payload): CreateOrderData
    {
        return CreateOrderData::from([
            'customerEmail'   => $payload['buyer']['email'],
            'notes'           => $payload['order_notes'] ?? null,
            'status'          => $payload['state'],
            'items'           => $payload['products'],
            'shippingAddress' => $payload['delivery'],
        ]);
    }
}
```

**When to use a transformer vs a static factory method:**

| Use static method on DTO | Use transformer class |
|---|---|
| Single source (one request type) | Multiple sources map to same DTO |
| Simple field mapping | Complex transformation logic |
| Small codebase or early stage | Transformation needs its own tests |

---

## Model JSON Column Casts

Cast JSON columns to DTOs directly on the model:

```php
// In the model
protected function casts(): array
{
    return [
        'metadata' => OrderMetadataData::class,
        'status'   => OrderStatus::class,
    ];
}

// Writing
$order->update(['metadata' => $metadataData]);

// Reading — returns OrderMetadataData instance
$label = $order->metadata->shippingLabel;
```

---

## Using DTOs in Controllers

Controllers pass DTOs into services. The service receives a typed DTO, not raw request data.

**For API responses:** Return Data objects directly - they automatically serialize to JSON. See the `laravel-api` skill for full API response conventions and patterns.

```php
class OrdersController extends Controller
{
    public function store(
        CreateOrderRequest $request,
        OrderService $orderService,
    ): JsonResponse {
        $data = CreateOrderData::fromRequest($request);

        $order = $orderService->create($data);

        return response()->json(OrderData::from($order), 201);
    }
}
```

---

## Using DTOs in Services

Services accept DTOs as parameters, not raw arrays or primitives:

```php
class OrderService
{
    public function create(CreateOrderData $data): Order
    {
        return \DB::transaction(function () use ($data) {
            $order = Order::create([
                'customer_email' => $data->customerEmail,
                'notes'          => $data->notes,
                'status'         => $data->status,
            ]);

            foreach ($data->items as $itemData) {
                $order->items()->create([
                    'product_id' => $itemData->productId,
                    'quantity'   => $itemData->quantity,
                ]);
            }

            return $order;
        });
    }
}
```

---

## Directory Structure

```
app/Data/
├── CreateOrderData.php
├── UpdateOrderData.php
├── OrderData.php
├── OrderItemData.php
├── ShippingAddressData.php
├── Formatters/
│   ├── EmailFormatter.php
│   └── PhoneFormatter.php
└── Transformers/
    └── Api/
        └── V1/
            └── OrderDataTransformer.php
```

---

## Testing DTOs

Use camelCase method names. Test construction, formatting, and casting:

```php
class CreateOrderDataTest extends TestCase
{
    public function testFromArrayCastsTypesCorrectly(): void
    {
        $data = CreateOrderData::from([
            'customerEmail' => 'test@example.com',
            'status'        => 'pending',
            'deliveryDate'  => '2026-03-01 00:00:00',
            'items'         => [],
            'shippingAddress' => ['line1' => '1 Main St', 'city' => 'Auckland'],
        ]);

        $this->assertSame('test@example.com', $data->customerEmail);
        $this->assertEquals(OrderStatus::Pending, $data->status);
        $this->assertInstanceOf(CarbonImmutable::class, $data->deliveryDate);
    }

    public function testFormatterNormalisesEmail(): void
    {
        $data = new CreateOrderData(
            customerEmail: '  TEST@EXAMPLE.COM  ',
            notes: null,
            deliveryDate: null,
            status: OrderStatus::Pending,
            items: \collect([]),
            shippingAddress: new ShippingAddressData(line1: '1 Main St', city: 'Auckland'),
        );

        $this->assertSame('test@example.com', $data->customerEmail);
    }
}
```

---

## Quick Reference

```
Primitive values between layers?    → Wrap in a DTO
Creating from external input?       → ::from() with array
Already have correct types?         → new DTO(...)
One source, simple mapping?         → Static fromRequest() on DTO
Multiple sources or complex?        → Dedicated transformer class
JSON column on model?               → Cast to DTO in casts()
Need normalisation?                 → Formatter in constructor
Paginated API response?             → ::collect($paginator, PaginatedDataCollection::class)
                                      (See laravel-api skill for full API patterns)
```
