---

title: "Features"
date: 2020-10-19T08:21:22Z
draft: false
weight: 6
hide: ["header"]

---

Represents a human-readable project feature in a class, named the way you would describe it to your colleagues and clients.
It contains the logic of implementing the feature with minimum friction and level of detail to remain concise and straight to the point.

It runs Lucid Units: *Jobs* and *Operations* to perform its tasks. They are thought of as the steps in the process of serving its purpose. A *Feature* can be served from anywhere, most commonly Controllers and Commands. Can also be queued to run asynchronously using Laravel's powerful [Queueing capabilities](https://laravel.com/docs/queues).

---

**Technically**, it is a class that encapsulates all the functionalities required for a single request/response lifecycle (or command), in which the `handle` method represents the task list when you want to implement it in your application.

**Example:** This is how a Feature class typically looks like. *(simplified)*
```php
class UpdateProductFeature extends Feature
{
    private string $id;

    public function __construct(string $id)
    {
        $this->id = $id;
    }

    public function handle(Request $request)
    {
        $this->run(ValidateProductInputJob::class, $request->input());

        $product = $this->run(SaveProductJob::class, [
            'id' => $this->id,
            'title' => $request->input('title'),
            'price' => $request->input('price'),
        ]);

        return $this->run(new RespondWithJsonJob($product));
    }
}

```

{{% notice info %}}
{{% icon name="fa-info-circle" %}}&nbsp;Running Lucid Units asynchronously is as simple as having them implement the `Queueable` interface, applies to all of
units such as *Jobs* and *Operations* and *Features*.
{{% /notice %}}

{{% figure alt="Lucid Feature Definition" src="/media/images/feature/controller-feature.png" width="600px" %}}
**Responsibility:** Perform the steps required to accomplish the feature by running *Jobs* and *Operations*.

```php
class UserController extends Controller
{
    public function login()
    {
        return $this->serve(LoginUserFeature::class);
    }
}
```

## Generate Feature Class
Use `lucid` CLI to generate a Feature class that extends Lucid's Feature base class by default, which allows us to run jobs and operations using the built-in `run` method.


{{% tabs %}}

    {{% tab "Micro" %}}
**Signature**
```bash
 lucid make:feature <feature>
```

**Example**
```bash
 lucid make:feature ListProducts
```
Generated class will be at `app/Features/ListProductsFeatures.php`

and its test at `tests/Features/ListProductsFeaturesTest.php`
    {{% /tab %}}


    {{% tab "Monolith" %}}
**Signature**
```bash
 lucid make:feature <feature> <service>
```

**Example**
```bash
 lucid make:feature ListProducts Commerce
```
Generated class will be at `src/Services/Commerce/Features/ListProductsFeatures.php`

and its test at `src/Services/Commerce/Tests/Features/ListProductsFeaturesTest.php`
    {{% /tab %}}

{{% /tabs %}}


The generated Feature class will automatically be suffixed with `Feature`, so the
class above will be `ListProductsFeature`.

{{% notice info %}}
{{% icon name="fa-asterisk" %}}&nbsp;For more details on this command see the help manual with `lucid make:feature --help`
{{% /notice %}}

## Inside Features

There are two essential highlights in a feature:
1. Signature (a.k.a constructor parameters): Looking at the signature of the *Feature* should give the idea of what is required for it to be served.
2. `handle` method: Looking at the class's `handle` method should provide an overview of the steps required to serve the feature to the user (or any consuming party) without having to know too much details about the inner workings of each step. More on this at [below](#the-handle-method).

{{% figure alt="Lucid Feature Expanded" src="/media/images/feature/controller-feature-expanded.png" width="1000px" %}}

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

{{% panel %}}
{{% icon name="fa-bolt" %}} It is recommended to always specify the required parameters in each unit's constructor rather than hiding them within
the unit. i.e. pass each input parameter separately rather than the entire `$request->input()`, or relying
on the unit to inject `Request` class. This way we keep it clear to know what is required for a job or operation to do its work.
{{% /panel %}}
## The `handle` method

Lucid units (Feature, Job, Operation) are simply classes extending Laravel's base Job class with extra functionality that ties them together. Hence, the use of `handle` method as the invocation point for each of these units.

```php
public function handle(Request $request)
```

This method is called automatically when running `$this->serve(Feature::class)` and it goes through Laravel's IoC to resolve dependencies. In this example we included the `Request` class to be resolved so that we can access it and pass input to *Jobs*. `Request` could've been any other class in the application that can be resolved using IoC.

```php
public function handle(MyCustomClass $mcc)
```

This is the recommended way of using classes to maintain testability by interchanging class instances with their mocks.

{{% notice info %}}
{{% icon name="fa-info-circle" %}}&nbsp;`handle` is the same for all Lucid units: *Feature, Job* and *Operation* and it behaves the same everywhere.
{{% /notice %}}

## Serving Features

You may serve features from anywhere in your application! Here are some exapmles of doing so.

### HTTP

If your controller is generated through the `lucid` command, all you need to do is call `serve` within the controller method,
otherwise, have your controller (or parent controller) extend Lucid's controller `Lucid\Foundation\Http\Controller`.

```php
use Lucid\Foundation\Http\Controller;
use App\Features\UpdateProductsFeature;

class ProductController extends Controller
{
    public function products()
    {
        return $this->serve(ListProductsFeature::class);
    }
}
```

### Command

To run a feature from the command we just need to equip our Command class with Lucid's methods by simply using `ServesFeaturesTrait`

```php
use Illuminate\Console\Command;
use App\Features\CleanStaleCartsFeature;
use Lucid\Foundation\ServesFeaturesTrait;

class CleanStaleCarts extends Command
{
    use ServesFeaturesTrait;

    protected $signature = 'clean:carts --stale';

    protected $description = "Cleans inactive customer carts.";

    public function handle()
    {
        return $this->serve(CleanStaleCartsFeature::class);
    }
}

```

### Other

Just like we did for [Commands](#command) we may do the same in any class we want, by simply using `ServesFeaturesTrait`
the `serve` method will be available.

```php
use App\Features\DoSomethingFeature;
use Lucid\Foundation\ServesFeaturesTrait;

class ToServeFeaturesHere
{
    use ServesFeaturesTrait;

    public function give()
    {
        return $this->serve(GiveHighFiveFeature::class);
    }
}
```

## Testing

When generating a feature with `lucid make:feature` a test class is automatically generated along, having `markIncomplete()` statement
in a stub method as a reminder to fill the missing test.

{{% tabs %}}

{{% tab "Micro" %}}

`lucid make:feature ListProductsFeature`

Would generate two files:
- `app/Features/ListProductsFeature`
- `tests/Features/ListProductsFeatureTest`

{{% /tab %}}

{{% tab "Monolith" %}}
`lucid make:feature ListProductsFeature api`

Would generate two files:
- `src/Services/Api/Features/ListProductsFeature`
- `src/Services/Api/Tests/Features/ListProductsFeatureTest`

Since Monolith is about scope and distribution of responsibility, and features are a scope of a service,
tests are distributed and scoped accordingly in the corresponding service.

{{% /tab %}}

{{% /tabs %}}

Feature tests are equivalent to functional tests in a typical application. It is about testing how the feature would behave
with a certain input combination from a user's perspective.

If the feature is served over an HTTP request, the test would be about actually requesting the URL and passing parameters
to receive the expected output, in addition to further assertions in the case of storage or other application aspects
that require further assertion.

{{% notice light %}}
The example below is incomplete (e.g. `$fake` must be defined) just to illustrate the concept.
{{% /notice %}}

```php
class UpdateProductDetailsFeatureTest extends TestCase
{
    public function test_successful_product_details_update()
    {
        $product = [
            'id' => $fake->uuid,
            'title' => $fake->sentence,
            'description' => $fake->text,
            'price' => $faker->randomNumber(2),
        ];

        // request
        $response = $this->put("/products/{$product['id']}", $product);
        $response->assertStatus(200)
                 ->assertJson([
                    'updated' => true
                 ]);

        // storage
        $stored = Product::find($production['id']);
        $this->assertEquals($product, $stored->toArray());
    }

    public function test_failing_product_details_update()
    {
        $product = [
            'id' => $fake->uuid,
            'title' => '', // cannot be empty
            'description' => $fake->text,
            'price' => $faker->randomNumber(2),
        ];

        $response = $this->put("/products/{$product['id']}", $product);
        $response->assertStatus(200)
                 ->assertJson([
                    'updated' => false,
                    'error' => [
                        'messages' => ['title field is required.'],
                    ],
                 ]);

        // storage - ensure original title is still in place
        $stored = Product::find($production['id']);
        $this->assertEquals('previous title', $stored->title);
    }
}
```

As far as functional testing goes, it is best to portray the usage of the feature from a broad perspective rather than digging
into every detail of the steps of the function. Feature testing is usually considered to be the integration test between
the units that make up the feature.

### Mocking

It is recommended with feature tests to only mock what's external to your application to preserve the quality guarantee .
Mocks are usually for 3rd-party APIs and other services that we integrate with, but **not** for internals such as storage.

**For this example we will assume that this feature runs from a command line rather than over HTTP.**

{{% notice light %}}
This example contains missing variables and is for illustrative purposes only.
{{% /notice %}}

`app/Features/UpdateFacebookPosts.php`

```php
class UpdateFacebookPosts extends Feature
{
    public function handle()
    {
        $posts = $this->run(FetchFacebookPostsJob::class);

        $this->run(StoreFacebookPostsJob::class, [
            'posts' => $posts,
        ]);
    }
}
```

`app/Domains/Facebook/Jobs/FetchFacebookPostsJob.php`
```php
class FetchFacebookPostsJob extends Job
{
    public function handle(FacebookClient $fb)
    {
        return $fb->posts();
    }
}
```

`tests/Features/UpdateFacebookPostsTest.php`
```php

class UpdateFacebookPostsTest extends TestCase
{
    public function test_successfully_fetching_facebook_posts()
    {
        $mFB = Mockery::mock(FacebookClient::class);
        $mFB->shouldReceive('get')->with($args)->andReturn($payload);

        $this->app->instance(FacebookClient::class, $mFB);

        $f = new UpdateFacebookPosts();
        $result = $f->handle();

        // 0 in CLI means success!
        $this->assertEquals(0, $result);
    }
}
```

As shown in `FetchFacebookPostsJob` the client has been injected, so we had to replace the instance in IoC to load our mocked
instance instead of the real one.

---

## FAQ
{{% panel %}}

### What not to do in a feature?

- **Complex conditional logic**: The feature passes output from *Jobs* and *Operations* but it barely knows anything about what goes on inside them. It only knows the sequence in which they should run and [maybe] some basic logic that is better be avoided and delegated to jobs and operations of possible.
- **Process output from units**: Parsing and transformation of content is best done in *Job* or an *Operation*. The output returned by a these units should always go as-is to the next unit.
The feature doesn't know much about the internals of steps or how they are actually performed. It is only aware of
the sequence of the steps and the requirements of the different units through their signatures.
This guarantees consistency and predictability when reading the feature's code in the `handle` method and allows anyone new
to the code to skim through quickly and have an idea about the required steps.
- **Call another Feature:** In order to avoid the [Pyramid of Doom](https://en.wikipedia.org/wiki/Pyramid_of_doom_(programming)) and reduce the [cognitive load](https://en.wikiversity.org/wiki/Software_Design/Cognitive_load) required to navigate the codebase and understand the code, Features should not call each other. Though the same feature may be called numerous times as long as
it does not add up to the Feature's [complexity](https://en.wikiversity.org/wiki/Software_Design/Complexity_(primary_quality)).

{{% /panel %}}

{{% panel %}}
### Should a Feature pass the entire set of input to other units?

More elaborately: instead of specifying parameters in constructors, jobs and operations would accept an associative array of input
and operate on that instead of a concise set of parameters.

Another option from

For several reasons:

- **Clarity:**
- **Isolation**:
- **Reusability:**

{{% /panel %}}

{{% panel %}}
### [Monolith] What to do about duplicate functionality between services?

{{% notice dark %}}
TL;DR It is recommended to have similar features across services than share features between them.
Even if it meant repeating the same sequence of units.
{{% /notice %}}

Since Monolith is all about separation of concerns per service, when dealing with multiple sides of the same application
(e.g. Api, CMS, Web UI, etc.) we'd need to deal with their features differently; maybe not at the beginning but eventually
we'll find ourselves dealing with too many options and cramming a lot of confusing parameters in the same feature
because we decided to share it with multiple services. To avoid all this, it is best to create a feature per service
even if it means to repeat the same sequence of units. Below is a brief example to illustrate it.

**Shared Feature**

Supposing that we decided to move the feature class from `src/Services/{service}/Features` to a place all services may access
like `app/Features/UpdateProductDetailsFeature.php` and we'd like our Api and Web services to use it.
This feature should serve the response in JSON when in the Api service and return a view when in Web:

<div style="margin-left: 23px;">

```php
class UpdateProductDetailsFeature
{
    private $isApi;

    public function __construct(bool $isApi)
    {
        $this->isApi = $isApi;
    }

    public function handle(Request $request)
    {
        $this->run(ValidateProductDeatilsInputJob::class, [
            'input' => $request->input()
        ]);

        $product = $this->run(UpdateProductDetailsJob::class, [
            'title' => $request->input('title'),
            'description' => $request->input('description'),
            'price' => $request->input('price'),
        ]);

        if ($this->isApi) {
            return $this->run(new RespondWithJsonJob($product));
        }

        return $this->run(RespondWithViewJob::class, [
            'view' => 'product',
            'data' => compact('product'),
        ]);
    }
}
```
</div>

This is yet a simple example but if we were to add more conditions to customize the experience further, we'd be throwing
plenty of `$this->isApi` everywhere in our feature. For example: the user reference from the API is a token, while the one
from the Web is an ID retrieved from the session. Here's how it would look like:

<div style="margin-left: 23px;">

```php
public function handle(Request $request)
{
    $this->run(ValidateProductDeatilsInputJob::class, [
        'input' => $request->input()
    ]);

    if ($this->isApi) {
        $user = $this->run(GetUserByApiTokenJob::class, [
            'token' => $request->input('token'),
        ]);
    } else {
        $user = $this->run(GetUserFromSessionJob::class);
    }

    $product = $this->run(UpdateProductDetailsJob::class, [
        'title' => $request->input('title'),
        'description' => $request->input('description'),
        'price' => $request->input('price'),
        'user' => $user,
    ]);

    if ($this->isApi) {
        return $this->run(new RespondWithJsonJob($product));
    }

    return $this->run(RespondWithViewJob::class, [
        'view' => 'product',
        'data' => compact('product'),
    ]);
}
```

</div>

Ugly, isn't it? See the other example for a cleaner approach below.

**Feature per Service**

`Api::UpdateProductDetailsFeature`

located at `src/Services/Api/Features/UpdateProductDetailsFeature.php`

<div style="margin-left: 23px;">

```php
class UpdateProductDetailsFeature
{
    public function handle(Request $request)
    {
        $this->run(ValidateProductDeatilsInputJob::class, [
            'input' => $request->input()
        ]);

        $user = $this->run(GetUserFromSessionJob::class);

        $product = $this->run(UpdateProductDetailsJob::class, [
            'title' => $request->input('title'),
            'description' => $request->input('description'),
            'price' => $request->input('price'),
            'user' => $user,
        ]);

        return $this->run(RespondWithViewJob::class, [
            'data' => compact('product'),
            'view' => 'web.product.update',
        ]);
    }
}
```

</div>

`Web::UpdateProductDetailsFeature`

located at `src/Services/Web/Features/UpdateProductDetailsFeature.php`

<div style="margin-left: 23px;">

```php
class UpdateProductDetailsFeature
{
    public function handle(Request $request)
    {
        $this->run(ValidateProductDeatilsInputJob::class, [
            'input' => $request->input()
        ]);

        $user = $this->run(GetUserByApiTokenJob::class, [
            'token' => $request->input('token'),
        ]);

        $product = $this->run(UpdateProductDetailsJob::class, [
            'title' => $request->input('title'),
            'description' => $request->input('description'),
            'price' => $request->input('price'),
            'user' => $user,
        ]);

        return $this->run(RespondWithViewJob::class, [
            'view' => 'product',
            'data' => compact('product'),
        ]);
    }
}
```

</div>

{{% /panel %}}
