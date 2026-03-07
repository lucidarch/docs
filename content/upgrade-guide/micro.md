---
title: "Micro"
date: 2020-12-30T18:12:56Z
draft: false
weight: 1
hide: ["header", "toc"]
---

## Upgrading to 2.0

**Requires Laravel 9+ and PHP 8.1+.**

### Update RouteServiceProvider in each service

The `$namespace` parameter has been removed from `loadRoutesFiles()`. Open every
`app/Services/<Service>/Providers/RouteServiceProvider.php` and remove the namespace argument:

```php
// Before
public function map(Router $router)
{
    $this->loadRoutesFiles($router, 'App\Services\MyService\Http\Controllers', $pathApi, $pathWeb);
}

// After
public function map(Router $router)
{
    $this->loadRoutesFiles($router, $pathApi, $pathWeb);
}
```

### Update route files to use fully-qualified controller class names

```php
// Before
Route::get('/', 'MyController@index');

// After
use App\Services\MyService\Http\Controllers\MyController;
Route::get('/', [MyController::class, 'index']);
```

---

## Upgrading from lucid-arch/* to lucidarch/lucid

Ensure that your tests are passing at the initial state of the upgrade to be able to compare with by the end that you've received the expected result.

- remove deprecated Lucid packages

    `composer remove lucid-arch/laravel-foundation lucid-arch/laravel-console`

    It is expected to see failures in the post-script since there are classes that no longer exist

- add `lucidarch/lucid`

    `composer require lucidarch/lucid`

- Replace unit namespaces (find & replace in `app` folder. Additionally you may want to search other folders such as `tests`):
    - Jobs

        `use Lucid\Foundation\Job;` → `use Lucid\Units\Job;`

    - Operations

        `use Lucid\Foundation\Operation;` → `use Lucid\Units\Operation;`

    - Features

        `use Lucid\Foundation\Feature;` → `use Lucid\Units\Feature;`

    - Controllers

        `use Lucid\Foundation\Http\Controller;` → `use Lucid\Units\Controller;`

    - Exceptions

        `Lucid\Foundation\InvalidInputException` → `Lucid\Exceptions\InvalidInputException`

    - Validator

        `use Lucid\Foundation\Validator;` → `use Lucid\Validation\Validator;`

    - Validation

        `use Lucid\Foundation\Validation;` → `use Lucid\Validation\Validation;`

    - Events

        `use Lucid\Foundation\Events` → `use Lucid\Events` (without `;`)

- Replace traits namespaces
    - `use Lucid\Foundation\ServesFeaturesTrait;` → `use Lucid\Bus\ServesFeatures;`
    - `use ServesFeaturesTrait;` → `use ServesFeatures;`

    - `use Lucid\Foundation\MarshalTrait;` → `use Lucid\Bus\Marshal;`
    - `use MarshalTrait;` → `use Marshal;`
- Run `composer dump-autoload` to feel the waters

    This time it should all be clear of errors. Otherwise, keep digging and replacing until it passes

- Run tests again and expect the same results as in the run before the upgrade
