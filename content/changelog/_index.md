---
title: "Changelog"
date: 2026-03-07T00:00:00Z
draft: false
weight: 100
hide: ["header"]
---

## v2.0.0

### Breaking Changes

**`RouteServiceProvider::loadRoutesFiles()` — `$namespace` parameter removed**

The `$namespace` parameter has been removed from `loadRoutesFiles()`, `mapApiRoutes()`, and `mapWebRoutes()`. Laravel 9 removed support for route group namespaces.

Update all generated `RouteServiceProvider.php` files in your services:

```php
// Before (Lucid 1.x)
public function map(Router $router)
{
    $this->loadRoutesFiles($router, 'App\Services\MyService\Http\Controllers', $pathApi, $pathWeb);
}

// After (Lucid 2.0)
public function map(Router $router)
{
    $this->loadRoutesFiles($router, $pathApi, $pathWeb);
}
```

Route files must reference controllers by fully-qualified class name:

```php
// Before
Route::get('/', 'MyController@index');

// After (Laravel 9+)
use App\Services\MyService\Http\Controllers\MyController;
Route::get('/', [MyController::class, 'index']);
```

**`RouteServiceProvider::mapApiRoutes()` and `mapWebRoutes()` — `$namespace` parameter removed**

These are protected methods; only affects subclasses that override them. Remove the `$namespace` argument from any overrides.

**PHP minimum raised to `^8.1`**

PHP 7.x and 8.0 are no longer supported.

**Laravel minimum raised to 9**

Laravel 5.x through 8.x are no longer supported. Minimum supported version is Laravel 9.

---

### Fixed

- **`Command::secret()`** — replaced removed Symfony Dialog helper with `QuestionHelper`. This method was broken for all users running Laravel 9+ (Symfony 6).
- **`Command::ask()`** — was incorrectly using `ConfirmationQuestion` (yes/no only); now uses `Question` for free-text input.

### Changed

- **`src/Units/Model.php`** — now explicitly imports `Illuminate\Database\Eloquent\Model` instead of relying on the global `Eloquent` alias.
- **`src/Bus/UnitDispatcher.php`** — now explicitly imports `Illuminate\Support\Facades\App` instead of relying on the global `App` alias.
- **`src/Console/Command.php`** — `call_user_func_array()` replaced with spread operator for `addArgument` and `addOption` calls.
- **Symfony packages** — pinned to `^5.4|^6.0|^7.0` (was wildcard `*`).
- **`mockery/mockery`** — moved from `require` to `require-dev`.
- **`minimum-stability`** — changed from `dev` to `stable`.
