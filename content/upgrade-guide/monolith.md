---
title: "Monolith"
date: 2020-12-30T18:12:59Z
draft: false
weight: 2
hide: ["header", "toc"]
---

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

- `Services/<service>/Tests` → `tests/Services/<service>`
- Move domain tests `app/Domains/<domain>/Tests` → `tests/Domains/<domain>`

    use the script below to do that for you, run it at the root of the project:

    ```bash
    #!/bin/bash
    BASE_DIR=$(pwd)
    TESTS_DIR="$BASE_DIR/tests"

    mkdir $BASE_DIR/tests/Domains

    for d in ./app/Domains/* ; do
        DOMAIN=${d##*/}
        echo "moving $d/Tests --> $TESTS_DIR/Domains/$DOMAIN"
        mv "$d/Tests" $TESTS_DIR/Domains/$DOMAIN
    done

    mkdir $BASE_DIR/tests/Services

    for s in ./app/Services/* ; do
        SERVICE=${s##*/}
        echo "moving $s/Tests --> $TESTS_DIR/Services/$SERVICE"
        mv "$s/Tests" $TESTS_DIR/Services/$SERVICE
    done
    ```

## Namespaces

- Rename Framework namespaces in `/app`
    - `namespace Framework\` → `namespace App\`
    - `use Framework` → `use App\`
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

The only content that should be there (besides custom ones) is the following:

```json
"psr-4": {
    "App\\": "app/",
    "Tests\\": "tests/"
}
```
