---
title: "Monolith"
date: 2020-12-30T18:12:59Z
draft: false
weight: 2
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

## Files

Moving everything from `/src` to `/app`

- Data
- Domains
- Foundation
- Services

- `Services/<service>/Tests/Features` → `tests/Feature/Services/<service>`
- `Services/<service>/Tests/Operations` and other unit tests → `tests/Unit/Services/<service>/Operations`
- Move domain tests `app/Domains/<domain>/Tests` → `tests/Unit/Domains/<domain>`

    use the script below to do that for you, run it at the root of the project:

    ```bash
    #!/bin/bash
    BASE_DIR=$(pwd)
    TESTS_DIR="$BASE_DIR/tests"

    mkdir $BASE_DIR/tests/Unit/Domains

    for d in ./app/Domains/* ; do
        DOMAIN=${d##*/}
        echo "moving $d/Tests --> $TESTS_DIR/Unit/Domains/$DOMAIN"
        mv "$d/Tests" $TESTS_DIR/Unit/Domains/$DOMAIN
    done

    mkdir $BASE_DIR/tests/Unit/Services $BASE_DIR/tests/Feature/Services

    for s in ./app/Services/* ; do
        SERVICE=${s##*/}
        echo "moving $s/Tests/Features --> $TESTS_DIR/Feature/Services/$SERVICE"
        mv "$s/Tests/Features" $TESTS_DIR/Feature/Services/$SERVICE

        echo "moving $s/Tests/Operations --> $TESTS_DIR/Unit/Services/$SERVICE/Operations"
        mv "$s/Tests/Operations" $TESTS_DIR/Unit/Services/$SERVICE/Operations
    done
    ```

## Namespaces

- Rename Framework namespaces in `/app`
    - `namespace Framework\` → `namespace App\`
    - `use Framework\` → `use App\`
- Rename Framework in `/bootstrap` , `/app` and `/config`
    - `Framework\` → `App\`
- Rename custom namespace in `/app`
    - `namespace SocialSource\` → `namespace App\`
    - `use SocialSource\` → `use App\`
- `app/RouteServiceProvider.php` rename `Framework\` to `App\` in `protected $namespace`

Replace unit namespaces (find & replace in `app` folder. Additionally you may want to search other folders such as `tests`):

- Jobs

    `use Lucid\Foundation\Job;` → `use Lucid\Units\Job;`

- Operations

    `use Lucid\Foundation\Operation;` → `use Lucid\Units\Operation;`

- Features

    `use Lucid\Foundation\Feature;` → `use Lucid\Units\Feature;`

- Services

    `Lucid\Foundation\Providers` → `Lucid\Providers`

- Controllers

    `use Lucid\Foundation\Http\Controller;` → `use Lucid\Units\Controller;`

- Exceptions

    `Lucid\Foundation\InvalidInputException` → `Lucid\Exceptions\InvalidInputException`

- Validator

    `use Lucid\Foundation\Validator;` → `use Lucid\Validation\Validator;`

- Replace traits namespaces
    - `use Lucid\Foundation\ServesFeaturesTrait;` → `use Lucid\Bus\ServesFeatures;`
    - `use ServesFeaturesTrait;` → `use ServesFeatures;`
- Update `phpunit.xml`
    - Remove `Services` and `Domains` testsuits

## Composer

- delete the line under `autoload.psr-4`: `"Framework\\": "app/",`
- reset namespace of `src` `"App\\": "app/",`

The following content (besides custom ones) must be present in `composer.json`:

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "app/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\": "tests/"
        }
    }
}
```

- Run `composer dump-autoload` to feel the waters

    This time it should all be clear of errors. Otherwise, keep digging and replacing until it passes

- Run tests again and expect the same results as in the run before the upgrade
