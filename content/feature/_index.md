---
title: "Feature"
date: 2020-10-19T08:21:22Z
draft: true
weight: 6
hide: ["toc"]
---

Represents a human-readable project feature in a class, named the way you would describe it to your colleagues and clients.
It contains the logic of implementing the feature with minimum friction and level of detail to remain concise and straight to the point.

It runs Lucid Units: *Jobs* and *Operations* to perform its tasks. They are thought of as the steps in the process of serving its purpose. A *Feature* can be served from anywhere, most commonly Controllers and Commands. Can also be queued to run asynchronously using Laravel's powerful [Queueing capabilities](https://laravel.com/docs/queues).

{{% notice %}}
{{% icon name="fa-info" %}} Running Lucid Units asynchronously is as simple as having them implement the `Queueable` interface, applies to all of
units such as *Jobs* and *Operations* and *Features*.
{{% /notice %}}

**Responsibility:** Perform the steps required to accomplish the feature by running *Jobs* and *Operations*.

What you should NOT do in a feature:

- Complex conditional logic: The feature passes output from *Jobs* and *Operations* but it barely knows anything about what goes on inside them. It only knows the sequence in which they should run and [maybe] some basic logic that is better be avoided and delegated to jobs and operations of possible.
- Parse or transform output from components: the output from one should always go as-is to the next. This guarantees consistency and predictability when reading the feature's code. If parsing or transformation is needed use a *Job* or an *Operation* to do that.
- Call another Feature.

{{% figure alt="Lucid Feature Definition" src="/media/images/feature/controller-feature.png" %}}

```php
class UserController extends Controller
{
    public function login()
    {
        return $this->serve(LoginUserFeature::class);
    }
}
```

## Inside Features

Think of a *Feature* as your task list when you want to implement it in the application, looking at the class's `handle` method should provide an overview of the steps required to serve the feature to the user (or any consuming party) without having to know too much details about the inner workings of each step. Also looking at the signature of the *Feature* should give the idea of what is required for it to be served.

![Definitions%20%E2%80%A2%20Components%206afacdab5e904deabc3fa4d1795bd08b/Untitled%203.png](Definitions%20%E2%80%A2%20Components%206afacdab5e904deabc3fa4d1795bd08b/Untitled%203.png)

```php
class CreateArticleFeature extends Feature
{
    public function handle(Request $request)
    {
        $this->run(new ValidateArticleInputJob($request->input()));

        $this->run(UploadFilesToCDNJob::class, ['files' => $request->input('files')]);

        $slug = $this->run(new GenerateSlugJob($request->input('title')));

        $article = $this->run(SaveArticleJob::class,
            [
                'title' => $request->input('title'),
                'body' => $request->input('body'),
                'slug' => $slug,
            ]
        );

        return $this->run(new RespondWithJsonJob($article));
    }
}
```

As shown in the example above, there are several ways to run Jobs in features that are explained in details in the definition of *Jobs*.

Let's examine the code above in depth...

# The `handle` method

```php
public function handle(Request $request)
```

This method is called automatically when a running `$this->serve(Feature::class)` and it goes through Laravel's IoC to resolve dependencies. In this example we included the `Request` class to be resolved so that we can access it and pass input to *Jobs*. `Request` could've been any other class in the application that can be resolved using IoC. Example:

```php
public function handle(MyCustomClass $mcc)
```

This is the recommended way of using classes to maintain testability by interchanging class instances with their mocks.

The `handle` is common to all Lucid components: *Feature, Job*  and *Operation* and it behaves the same everywhere.
