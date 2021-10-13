---
title: "Micro"
date: 2020-12-30T18:12:56Z
draft: false
weight: 1
hide: ["header", "toc"]
---

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
