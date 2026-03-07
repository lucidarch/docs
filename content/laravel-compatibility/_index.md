---
title: "Laravel Compatibility"
date: 2024-01-01T00:00:00Z
draft: false
weight: 6
head: "<hr />"
hide: ["header"]
---

Lucid Architecture is compatible with Laravel 9, 10, and 11 as well as PHP 8.1 through 8.4. This page documents
the key changes across these versions and how they relate to your Lucid application.

## Laravel 11

Laravel 11 introduced a streamlined application structure. Here are the key changes and how they affect Lucid projects.

### Simplified App Structure

Laravel 11 removed several files and consolidated others:

- **No more `app/Http/Kernel.php`** — middleware is registered in `bootstrap/app.php`
- **No more `app/Console/Kernel.php`** — console commands are registered in `routes/console.php`
- **No more `app/Exceptions/Handler.php`** — exception handling is in `bootstrap/app.php`
- **Single `routes/web.php` and `routes/api.php`** remain

**Lucid units (Jobs, Features, Operations) are unaffected** by these changes since they live in `app/Domains`, `app/Features`,
`app/Operations`, and `app/Services` — directories that Lucid itself manages.

### Exception Handling in Laravel 11

Previously, exception handling with custom dispatchers was placed in `app/Exceptions/Handler.php`.
In Laravel 11, register exception rendering in `bootstrap/app.php`:

```php
// bootstrap/app.php
use App\Exceptions\CustomException;
use Lucid\Bus\UnitDispatcher;
use App\Domains\Http\Jobs\RespondWithJsonErrorJob;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (CustomException $e, Request $request) {
        // Use the UnitDispatcher trait in a closure-compatible way via a helper class
        // or dispatch jobs directly from a dedicated error-handling service
        return response()->json([
            'status' => 500,
            'error' => ['message' => $e->getMessage()],
        ], 500);
    });
})
```

{{% notice info %}}
{{<icon name="fa-info-circle">}}&nbsp;For centralised JSON error responses in Laravel 11, create a dedicated exception handler class that uses
`UnitDispatcher` and register it in `bootstrap/app.php` to maintain the Lucid pattern.
{{% /notice %}}

### Middleware Registration

Middleware previously registered in `Http/Kernel.php` is now in `bootstrap/app.php`:

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->api(prepend: [
        \App\Http\Middleware\EnsureTokenIsValid::class,
    ]);
})
```

Lucid controllers and features that rely on middleware continue to work the same way since middleware is applied
at the routing layer, not within Lucid units.

### Vite Instead of Mix

Laravel 9 replaced Laravel Mix with **Vite** as the default asset bundler. The `webpack.mix.js` file is gone; it is now `vite.config.js`.

If you are upgrading a Lucid Monolith app from Laravel 8 to 9+, update your service providers' resource publishing accordingly and replace any Mix helper calls in Blade views:

```blade
{{-- Before (Mix) --}}
<link rel="stylesheet" href="{{ mix('css/app.css') }}">
<script src="{{ mix('js/app.js') }}"></script>

{{-- After (Vite) --}}
@vite(['resources/css/app.css', 'resources/js/app.js'])
```

---

## Laravel 10

Laravel 10 introduced native type declarations across the framework's stubs and generated files. New generated classes
(controllers, jobs, etc.) include return type hints.

When using Lucid's `lucid make:*` generators, the generated stubs follow these conventions. If you've customized
your Lucid stubs, ensure they include appropriate return types.

### `password_resets` → `password_reset_tokens`

Laravel 10 renamed the `password_resets` table to `password_reset_tokens`. If you are migrating from Laravel 8/9,
update your migration files accordingly.

---

## Laravel 9

Laravel 9 introduced several improvements relevant to Lucid applications:

- **PHP 8.0+ required** — enums, named arguments, and constructor promotion work in all Lucid units
- **Symfony 6 components** — upgraded HTTP kernel internals; no Lucid-specific changes needed
- **`route:list` improvements** — cleaner output when listing routes served by Lucid controllers

---

## PHP 8.x Features in Lucid

Modern PHP features make Lucid units more expressive and concise.

### Constructor Property Promotion (PHP 8.0+)

Eliminate boilerplate in Job constructors:

```php
// Before
class SaveProductJob extends Job
{
    private string $title;
    private float $price;
    private string $description;

    public function __construct(string $title, float $price, string $description)
    {
        $this->title = $title;
        $this->price = $price;
        $this->description = $description;
    }
}

// After (PHP 8.0+)
class SaveProductJob extends Job
{
    public function __construct(
        private string $title,
        private float $price,
        private string $description,
    ) {}
}
```

### Enums (PHP 8.1+)

Enums are a natural fit in the `Data` layer alongside models. Use them as Job parameters for type-safe options:

```php
// app/Data/Enums/OrderStatus.php
enum OrderStatus: string
{
    case Pending = 'pending';
    case Processing = 'processing';
    case Completed = 'completed';
    case Cancelled = 'cancelled';
}

// app/Domains/Order/Jobs/UpdateOrderStatusJob.php
class UpdateOrderStatusJob extends Job
{
    public function __construct(
        private int $orderId,
        private OrderStatus $status,
    ) {}

    public function handle(): Order
    {
        $order = Order::findOrFail($this->orderId);
        $order->update(['status' => $this->status->value]);
        return $order;
    }
}

// In a Feature
$this->run(new UpdateOrderStatusJob(
    orderId: $request->input('order_id'),
    status: OrderStatus::Completed,
));
```

### Readonly Properties (PHP 8.1+)

Jobs are naturally immutable — their state is set in the constructor and never modified. Use `readonly` to make this explicit:

```php
class GetUserByIdJob extends Job
{
    public function __construct(
        private readonly int $id,
    ) {}

    public function handle(): User
    {
        return User::findOrFail($this->id);
    }
}
```

### Named Arguments (PHP 8.0+)

Named arguments improve readability when calling jobs — this is already the recommended pattern in Lucid:

```php
$this->run(new CreateOrderJob(
    userId: $request->input('user_id'),
    items: $request->input('items'),
    shippingAddress: $request->input('address'),
));
```

### Intersection Types (PHP 8.1+)

Use intersection types in job `handle` method signatures for more precise dependency injection:

```php
public function handle(Cacheable&Repository $repository): void
{
    // ...
}
```

---

## Modern Laravel Integration

### Livewire Components

Livewire components can serve Lucid Features by using the `ServesFeatures` trait:

```php
use Livewire\Component;
use Lucid\Bus\ServesFeatures;
use App\Features\AddCommentFeature;

class CommentForm extends Component
{
    use ServesFeatures;

    public string $body = '';

    public function submit()
    {
        $this->validate(['body' => 'required|min:3']);

        $this->serve(new AddCommentFeature(
            body: $this->body,
            userId: auth()->id(),
        ));

        $this->reset('body');
    }

    public function render()
    {
        return view('livewire.comment-form');
    }
}
```

This keeps Livewire components thin: they handle UI state, validation, and delegation to Features — consistent with the Lucid philosophy.

### Laravel Octane

When running Lucid applications under [Laravel Octane](https://laravel.com/docs/octane), keep these guidelines in mind:

- **Jobs are stateless by design** — each Job instance is created fresh per request, making them safe under Octane's long-running process model.
- **Avoid storing request-scoped data in service container singletons** — if you bind custom classes as singletons in service providers, ensure they are re-bound per request using Octane's `RequestReceived` event or use `$this->app->scoped()` bindings.
- **Features and Operations follow the same rules** — since they delegate to Jobs, they inherit the same safety guarantees.

### Laravel Queues (Laravel 11)

Queueable Jobs in Lucid work with Laravel's full queue infrastructure. In Laravel 11, the queue configuration is in `config/queue.php` and the `ShouldQueue` contract is unchanged:

```php
use Illuminate\Contracts\Queue\ShouldQueue;
use Lucid\Units\Job;

class ProcessVideoUploadJob extends Job implements ShouldQueue
{
    public int $timeout = 300;
    public int $tries = 3;

    public function __construct(
        private readonly string $filePath,
        private readonly int $userId,
    ) {}

    public function handle(): void
    {
        // processing logic
    }
}
```

Use `lucid make:job ProcessVideoUpload files --queue` to generate the queueable stub automatically.
