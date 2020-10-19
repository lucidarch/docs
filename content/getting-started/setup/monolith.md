---
title: "Monolith"
date: 2020-10-13T14:54:00Z
draft: true
hide:
- toc
---

## Laravel • Lucid Monolith

This example uses Laravel with the Lucid Monolith installation (in contrast with Microservice). For more on the differences see Definitions#monolith-vs-microservice.

Initialise a Laravel project with Lucid, using the latest version:

```bash
composer create-project lucid-arch/laravel lucid-demo
```

This will install a new Laravel project with the latest version, structured as per the Lucid architecture, as well as the [Lucid Console](https://github.com/lucid-architecture/laravel-console) which provides a command line interface and a user interface to help you manage your Lucid components.

To install a specific version use notation such as `lucid-arch/laravel=5.8.x` instead of `lucid-arch/laravel`

### Directory Structure

What you should know about the directories that you will find in addition to the ones included in Laravel by default:

```bash
src
├── Data
├── Domains
    └── * domain name *
            ├── Jobs
├── Foundation
└── Services
    └── * service name *
        ├── Console
        ├── Features
        ├── Http
        ├── Providers
        ├── Tests
        ├── database
        └── resources
```

### Units

Use the table below to help you place the different Lucid units

|            | Path                                        | Description                                                                |
|------------|---------------------------------------------|----------------------------------------------------------------------------|
| Service    | `src/Service/{service}`                     | Place for the Services                                                     |
| Feature    | `src/Services/[service]/Features/[feature]` | Place for the Features of Services                                         |
| Job        | `src/Domains/[domain]/Jobs/[job]`           | Place for the Jobs that expose the functionalities of Domains              |
| Operation  | `src/Operations`                            | Place for the Operation that can call multiple Jobs from different domains |
| Data       | `src/Data`                                  | Place for models, repositories, value objects and anything data-related    |
| Foundation | `src/Foundation`                            | Place for foundational (abstract) elements used across the application     |

## Setup

The `lucid` executable will be in `vendor/bin`. If you don't have `./vendor/bin/` as part of your shell `PATH` you will need to execute it using `./vendor/bin/lucid`, otherwise add it with the following command to be able to simply call `lucid` when in the project's root directory [recommended]:

```bash
export PATH="./vendor/bin:$PATH"
```

Seek your shell session's docs to see how to add this to your shell profile permanently.

For a list of all the commands that are available run `lucid` or see the [CLI Reference](https://www.notion.so/mulkave/Getting-Started-66540e36f2d14b7ea0d9befe9d554df9).

## Description

Before we dive into code, here's what we are about to do:

We will create an application with basic functionality to create and list articles, where an article simply having `title` and `content`. The application will have two services: `Api`  and `Admin` . Each will expose the features correspondingly, where the `Api` service will receive and return JSON data, while `Admin` will serve them rendered using [Blade](https://laravel.com/docs/5.8/blade).
