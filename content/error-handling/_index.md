---
title: "Error Handling"
date: 2020-10-22T06:37:52Z
draft: false
weight: 20
hide: ["header"]

---

Features typically expose functionality to the outside world, meaning they have contracts to fulfill when returning results — whether JSON, Views (HTML), or other formats. Centralizing error handling in features guarantees consistency in those results.

The goal is to keep error handling minimal within each feature: handle only the exceptions that are specific to it, and delegate everything else to Laravel's central `Handler` class (for HTTP) or the caller in other contexts.

## Expected Exceptions

Expected exceptions are those that are specific to a given context and require a deliberate response. They arise from domain logic — a record that doesn't exist, a business rule violation, an authorization failure — and handling them explicitly is the right call.

The right place to handle these is in the feature that serves the user-facing action, or in the job where the exception originates.

**Handling in a Feature**

```php
class UpdateArticleFeature extends Feature
{
    public function __construct(private int $id) {}

    public function handle(Request $request)
    {
        try {
            $article = $this->run(new FindArticleByIDJob(id: $this->id));
        } catch (ArticleNotFoundException $e) {
            return $this->run(new RespondWithJsonErrorJob(
                status: 404,
                message: $e->getMessage(),
            ));
        }

        $this->run(new ValidateArticleInputJob($request->input()));

        $updated = $this->run(new UpdateArticleJob(
            id: $this->id,
            title: $request->input('title'),
            body: $request->input('body'),
        ));

        return $this->run(new RespondWithJsonJob($updated));
    }
}
```

Here `ArticleNotFoundException` is expected — the caller asked for a specific resource and it may or may not exist — so we handle it where the response contract is defined: in the feature.

**Handling in a Job**

Sometimes it makes more sense to handle an exception within the job itself, particularly when a job wraps a third-party integration or a domain operation with a well-defined failure mode:

```php
class FetchPaymentStatusJob extends Job
{
    public function __construct(private string $transactionId) {}

    public function handle(PaymentGateway $gateway): PaymentStatus
    {
        try {
            return $gateway->status($this->transactionId);
        } catch (GatewayTimeoutException $e) {
            // return a neutral status rather than crashing the feature
            return PaymentStatus::unknown();
        }
    }
}
```

{{% notice info %}}
{{% icon name="fa-info-circle" %}}&nbsp;Only handle exceptions you know how to recover from. Catching everything indiscriminately hides bugs and makes failures silent.
{{% /notice %}}

## Unexpected Exceptions

Unexpected exceptions are those that have no meaningful recovery path within a specific feature or job — database connectivity failures, unowned third-party timeouts, configuration errors. These are best left for Laravel's central `Handler` class.

Keeping unexpected errors out of individual features ensures that every unexpected failure is handled consistently across the entire application, without duplicating fallback behavior in each feature.

Laravel's `Handler` class is the right place to define catch-all behavior per exception type. Since any class can be turned into a unit dispatcher by using `UnitDispatcher`, the `Handler` can run jobs directly and leverage Lucid's infrastructure to produce consistent responses.

## The Exception Handler

{{% tabs %}}

{{% tab "Laravel 9 / 10" %}}

Equip your `Handler` class with `UnitDispatcher` to dispatch jobs from within it:

```php
namespace App\Exceptions;

use Lucid\Bus\UnitDispatcher;
use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;

class Handler extends ExceptionHandler
{
    use UnitDispatcher;
}
```

Then register renderable callbacks in the `register` method for each exception type you want to handle centrally.

**JSON API response**

```php
use App\Exceptions\PaymentFailedException;
use App\Domains\Http\Jobs\RespondWithJsonErrorJob;

public function register(): void
{
    $this->renderable(function (PaymentFailedException $e, $request) {
        return $this->run(new RespondWithJsonErrorJob(
            status: 402,
            message: $e->getMessage(),
        ));
    });
}
```

**HTML / View response**

```php
use App\Exceptions\MaintenanceModeException;
use App\Domains\Http\Jobs\RespondWithViewJob;

public function register(): void
{
    $this->renderable(function (MaintenanceModeException $e, $request) {
        return $this->run(new RespondWithViewJob(
            status: 503,
            template: 'errors.maintenance',
        ));
    });
}
```

{{% /tab %}}

{{% tab "Laravel 11" %}}

Laravel 11 removed `app/Exceptions/Handler.php`. Exception handling is now registered in `bootstrap/app.php` via `->withExceptions()`. Since closures cannot use traits directly, create a dedicated handler class that uses `UnitDispatcher`:

```php
// app/Exceptions/LucidExceptionHandler.php
namespace App\Exceptions;

use Lucid\Bus\UnitDispatcher;

class LucidExceptionHandler
{
    use UnitDispatcher;
}
```

Then resolve and use it from `bootstrap/app.php`:

```php
use App\Exceptions\LucidExceptionHandler;
use App\Exceptions\PaymentFailedException;
use App\Exceptions\MaintenanceModeException;
use App\Domains\Http\Jobs\RespondWithJsonErrorJob;
use App\Domains\Http\Jobs\RespondWithViewJob;
use Illuminate\Foundation\Application;

->withExceptions(function (Exceptions $exceptions) {
    $handler = app(LucidExceptionHandler::class);

    // JSON API response
    $exceptions->render(function (PaymentFailedException $e, $request) use ($handler) {
        return $handler->run(new RespondWithJsonErrorJob(
            status: 402,
            message: $e->getMessage(),
        ));
    });

    // HTML / View response
    $exceptions->render(function (MaintenanceModeException $e, $request) use ($handler) {
        return $handler->run(new RespondWithViewJob(
            status: 503,
            template: 'errors.maintenance',
        ));
    });
})
```

{{% notice info %}}
{{% icon name="fa-lightbulb" %}}&nbsp;In Laravel 11 it is often simpler to use [self-rendering exceptions](#self-rendering-exceptions) instead, since they carry their own response logic and require no central registration.
{{% /notice %}}

{{% /tab %}}

{{% /tabs %}}

The built-in `RespondWithJsonErrorJob` produces a consistent response envelope:

```json
{
    "status": 402,
    "error": {
        "code": 400,
        "message": "Payment method declined."
    }
}
```

{{% notice dark %}}
For more on registering exception renderers see [Laravel's exception handling docs](https://laravel.com/docs/errors#rendering-exceptions).
{{% /notice %}}

## Self-Rendering Exceptions

You may have exceptions render themselves by defining a `render` method directly on the exception class. Equipping the exception with `UnitDispatcher` allows it to dispatch jobs on its own:

```php
namespace App\Exceptions;

use Exception;
use Lucid\Bus\UnitDispatcher;
use App\Domains\Http\Jobs\RespondWithJsonErrorJob;

class InsufficientStockException extends Exception
{
    use UnitDispatcher;

    public function __construct(private int $productId, private int $requested)
    {
        parent::__construct("Insufficient stock for product #{$productId}: requested {$requested}.");
    }

    public function render($request)
    {
        return $this->run(new RespondWithJsonErrorJob(
            status: 422,
            message: $this->getMessage(),
        ));
    }
}
```

{{% panel %}}
{{% icon name="fa-lightbulb" %}}&nbsp;Self-rendering exceptions are particularly useful for domain-specific errors that always map to the same response, removing the need to register them in the `Handler` separately.
{{% /panel %}}

## Consistent Error Responses

Whether an error surfaces from a feature, a job, or the central handler, the goal is the same: a predictable structure the consumer can rely on.

The built-in `RespondWithJsonErrorJob` is available in the `Http` domain and accepts the following parameters:

```php
$this->run(new RespondWithJsonErrorJob(
    message: $e->getMessage(), // error message
    status: 400,               // HTTP status code (default: 400)
    code: 4001,                // internal error code, optional
    headers: [],               // additional response headers
    options: 0,                // JSON encoding options
));
```

This always returns the same envelope:

```json
{
    "status": 400,
    "error": {
        "code": 4001,
        "message": "A human-readable error message."
    }
}
```

Using this job consistently across features, operations, and the exception handler ensures your consumers — whether they're a frontend client, a mobile app, or another service — always receive the same structure regardless of where the failure occurred.

## Testing

Error paths are first-class scenarios that deserve their own tests. Feature tests are the most natural fit: they exercise the full path from input to response, including the branch where things go wrong.

**Testing an expected exception in a feature**

```php
class UpdateArticleFeatureTest extends TestCase
{
    public function test_returns_404_when_article_not_found()
    {
        $response = $this->putJson('/articles/9999', [
            'title' => 'New Title',
            'body'  => 'New Body',
        ]);

        $response->assertStatus(404)
                 ->assertJsonPath('error.message', 'Article not found.');
    }
}
```

**Testing that a job throws the expected exception**

```php
class FindArticleByIDJobTest extends TestCase
{
    public function test_throws_when_article_does_not_exist()
    {
        $this->expectException(ArticleNotFoundException::class);

        $job = new FindArticleByIDJob(id: 9999);
        $job->handle();
    }
}
```

{{% notice info %}}
{{% icon name="fa-info-circle" %}}&nbsp;Testing the happy path and the error path in separate test methods keeps each test focused and makes failure messages easy to understand.
{{% /notice %}}
