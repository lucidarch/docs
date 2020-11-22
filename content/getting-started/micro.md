---
title: "Micro"
date: 2020-11-15T11:20:52Z
draft: false
weight: 1
hide: ["header"]
---

{{% notice light %}}
<i class="fab fa-github fa-lg"></i>&nbsp;The source code for this exercise is on [GitHub](https://github.com/lucid-architecture/getting-started-micro).
{{% /notice %}}

In this guide we're going to build a link directory where we can register and save links of our own using Lucid Micro -
the default variant for single-purpose applications.

{{% notice dark %}}
{{<icon name="fa-archway">}} For Monolith - the multi-purpose service-oriented variant see [Getting Started • Monolith]({{<ref "/getting-started/monolith">}}).
{{% /notice %}}

From the Lucid stack we'd be using
[Features]({{<ref "/features">}}), [Jobs]({{<ref "/jobs">}}), [Domains]({{<ref "/domains">}}) and [Requests]({{<ref "/validation#/#form-request-validation">}})
to implement the following:

- Create a form to submit new links.
- Validate the form.
- Insert the data into the database.

{{% notice info %}}
{{<icon name="fa-info-circle">}}&nbsp;This tutorial is based on the excellent Laravel Tutorial:
    [Step by Step Guide to Building Your First Laravel Application](https://laravel-news.com/your-first-laravel-application).
{{% /notice %}}

## Setup

### Install Laravel

Let's start by creating a new Laravel project. It is best if you refer to [Laravel's installation docs](https://laravel.com/docs/installation)
and choose your preferred way of installation, but here are the common ways to do it:

```bash
# via the installer
laravel new links

# via composer
composer create-project --prefer-dist laravel/laravel links
```

### Install Lucid

```bash
composer require lucidarch/lucid
```

### Database Configuration

Now that we have our project ready with a `.env` file we can configure the database connection.

For the brevity of this example
we will utilise SQLite as it requires the least steps to get going. This is surely not recommended in real apps
but will make very little difference since you can change the configuration and use your favourite database
without affecting the code.

1. Create database file `storage/app/database/database.sql`
    ```bash
    mkdir -p storage/app/database && touch storage/app/database/database.sql
    ```

2. Configure database
    `.env`
    ```ini
    DB_CONNECTION=sqlite
    DB_DATABASE={ABSOLUTE PATH}/storage/app/database/database.sql
    ```

3. Create tables
    ```bash
    php artisan migrate
    ```
    And the output should match this:
    ```
    Migration table created successfully.
    Migrating: 2014_10_12_000000_create_users_table
    Migrated:  2014_10_12_000000_create_users_table (4.11ms)
    Migrating: 2014_10_12_100000_create_password_resets_table
    Migrated:  2014_10_12_100000_create_password_resets_table (2.14ms)
    Migrating: 2019_08_19_000000_create_failed_jobs_table
    Migrated:  2019_08_19_000000_create_failed_jobs_table (2.54ms)
    ```

---

## Implementation

We will be using a handful of Laravel features to demonstrate how they organically fit within the Lucid Architecture due to its
timeless approach towards application structure.

Frontend technologies used in this guide:
- [TailwindCSS](https://tailwindcss.com) for styling
- [Laravel Blade](https://laravel.com/docs/blade) for templating

## Authentication

We will try to avoid re-inventing the wheel as much as possible by using [`laravel/breeze`](https://github.com/laravel/breeze)
to scaffold configuration and Auth routes, views and controllers styled with TailwindCSS.

```bash
composer require laravel/breeze --dev

php artisan breeze:install

npm install && npm run dev
```

Now to watch assets and build on change we may run `npm run watch`.

The resulting files are under `resources/views/auth` and the `welcome` page has been updated to look as follows:

![Welcome](/media/images/getting-started/welcome.png)

To create an account click on Register at the top right and enter your account details to be logged in to the dashboard:

![Register](/media/images/getting-started/register.png)

## Link Submission

### View

First, create a new route to serve our view in `routes/web.php`:

```php
Route::get('/submit', function() {
    return view('submit');
});
```

Next, we need to create the `submit.blade.php` template at `resources/views/submit.blade.php` with the following boilerplate
to submit a link with a title and a description:

```html
<x-app-layout>
    <x-slot name="header">
        Add Link
    </x-slot>

    <div class="flex items-center justify-center h-screen">
        <form action="/submit" method="post" class="w-full max-w-sm bg-white shadow-md rounded px-8 pt-6 pb-8">
            @csrf
            @if ($errors->any())
                <div class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 mt-2 mb-2 rounded relative" role="alert">
                    Please fix the following errors
                </div>
            @endif
            <div class="md:flex md:items-center mb-6">
                <div class="md:w-1/3">
                    <label class="block text-gray-500 font-bold md:text-right mb-1 md:mb-0 pr-4" for="title">
                        Title
                    </label>
                </div>
                <div class="md:w-2/3">
                    <input id="title" value="{{ old('title') }}" name="title" class="appearance-none border-2 @error('title') border-red-400 @else border-gray-200 @enderror rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-teal-500" type="text">
                    @error('title')
                    <p class="text-red-500 text-xs italic mt-2">{{ $message }}</p>
                    @enderror
                </div>
            </div>
            <div class="md:flex md:items-center mb-6">
                <div class="md:w-1/3">
                    <label class="block text-gray-500 font-bold md:text-right mb-1 md:mb-0 pr-4" for="url">
                        URL
                    </label>
                </div>
                <div class="md:w-2/3">
                    <input id="url" value="{{ old('url') }}" name="url" class="appearance-none border-2 @error('url') border-red-400 @else border-gray-200 @enderror rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-teal-500" type="text">
                    @error('url')
                    <p class="text-red-500 text-xs italic mt-2">{{ $message }}</p>
                    @enderror
                </div>
            </div>
            <div class="md:flex md:items-center mb-6">
                <div class="md:w-1/3">
                    <label for="description" class="block text-gray-500 font-bold md:text-right mb-1 md:mb-0 pr-4" for="desription">
                        Description
                    </label>
                </div>
                <div class="md:w-2/3">
                    <textarea id="description" name="description" class="appearance-none border-2 @error('description') border-red-400 @else border-gray-200 @enderror rounded w-full py-2 px-4 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-teal-500" id="description" name="description">{{old('description')}}</textarea>
                    @error('description')
                    <p class="text-red-500 text-xs italic mt-2">{{ $message }}</p>
                    @enderror
                </div>
            </div>

            <div class="md:flex md:items-center">
                <div class="md:w-1/3"></div>
                <div class="md:w-2/3">
                    <button type="submit" class="shadow bg-teal-500 hover:bg-teal-400 focus:shadow-outline focus:outline-none text-white font-bold py-2 px-4 rounded">
                        Add
                    </button>
                </div>
            </div>
        </form>
    </div>
</x-app-layout>

```

![Submit Form](/media/images/getting-started/submit.png)

### Controller

Generate `LinkController` to manage our links using `lucid` and have it ready for serving features.

```bash
lucid make:controller Link
```

```bash
Controller class created successfully.

Find it at app/Http/Controllers/LinkController.php
```

{{% notice info %}}
Notice the automatic addition of `Controller` suffix, used as a naming convention to match other suffixes in the Lucid stack such as `Feature`, `Job` and `Operation`.
{{% /notice %}}

We will need a method to handle the form's submission request, let's call it `add`:

`app/Http/Controllers/LinkController.php`

```php
<?php

namespace App\Http\Controllers;

use Lucid\Units\Controller;

class LinkController extends Controller
{
    public function add()
    {

    }
}
```

### Feature

The `add` method will then serve the feature that will run the jobs required to add links.
Generate a feature called `AddLinkFeature`:

```bash
lucid make:feature AddLink
```

```bash
Feature class AddLinkFeature created successfully.

Find it at app/Features/AddLinkFeature.php
```

`add()` will now serve `AddLinkFeature`

`app/Http/Controllers/LinkController.php`

```php
use App\Features\AddLinkFeature;

...

public function add()
{
    return $this->serve(AddLinkFeature::class);
}
```

Beginning with the step of generating a feature will cognitively set our context to what we will be working on, helping us concentrate
on the task at hand.

Let's expose our feature with a route:

`routes/web.php`

```php
use App\Http\Controllers\LinkController;

Route::post('/submit', [LinkController::class, 'add']);
```

So far so good, now we'll fill our feature with the steps required to add a link.

### Database & Model

Before we can start accepting data we need to prepare our database with a migration to create the links table:

```bash
php artisan make:migration create_links_table --create=links
```

The generated file will contain the creation schema at `database/migrations/{datetime}_create_links_table.php`.
Let's add the fields for our link:

```php
public function up()
{
    Schema::create('links', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->string('url')->unique();
        $table->text('description');
        $table->timestamps();
    });
}
```

Run the migration to create the table in the database:

```bash
php artisan migrate
```

Then we create our `Link` model class at `app/Data/Models/Link.php`

```php
<?php

namespace App\Data\Models;

use Illuminate\Database\Eloquent\Model;

class Link extends Model
{
    protected $fillable = ['url', 'title', 'description'];
}
```

We're all set to start building our feature!

### Request Validation

The first step of receiving input is to validate it. We will be using [Form Request validation](https://laravel.com/docs/validation#form-request-validation)
where each Request belongs in a Domain representing the entity that's being managed, in this case it's `Link` containing an `AddLink` Request class.

This will be the beginning of working with Domains in Lucid. They're used to group Jobs and custom classes which logic is
associated with certain topic according to domain-driven design.

Starting with validation, Lucid places Request classes within their corresponding domains. Let's generate an `AddLink` request:

```bash
lucid make:request AddLink link
```

```bash
Request class created successfully.

Find it at app/Domains/Link/Requests/AddLink.php
```

In `AddLink` we'll need to update the methods `authorize()` and `rules()` to validate the request and its input:

```php
<?php

namespace App\Domains\Link\Requests;

use Illuminate\Support\Facades\Auth;
use Illuminate\Foundation\Http\FormRequest;

class AddLink extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return Auth::check();
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => ['required', 'max:255'],
            'url' => ['required', 'url', 'max:255'],
            'description' => ['required', 'max:255'],
        ];
    }
}
```

With our request ready, now we need `AddLinkFeature` to use that request class when served. We can do that by simply injecting
the request class in the feature's `handle()` method and every time the feature is served this validation will be applied.

```php
<?php

namespace App\Features;

use Lucid\Units\Feature;
use App\Domains\Link\Requests\AddLink;

class AddLinkFeature extends Feature
{
    public function handle(AddLink $request)
    {

    }
}

```

Now if we visit `/submit` and click `Add` wihtout passing any input it will generate errors and print their messages from validation failures.

![Validation errors UI](/media/images/getting-started/links-validation-errors.png)

### Save Links

To save the link we'll create a job that saves links and run it in our feature, which will be added to our `Link` domain at
`app/Domains/Link/Jobs/SaveLinkJob` along with its test `app/Domains/Link/Tests/Jobs/SaveLinkJobTest`.

```bash
lucid make:job SaveLik link
```

{{% notice info %}}
{{<icon name="fa-asterisk">}}&nbsp;Notice the naming that we've used with this job "`SaveLinkJob`" in contrast with "`AddLink`".
It is intended for reuse whenever needed by extending its functionality futher, for example `UpdateLink` feature may be able to use the same job.
{{% /notice %}}

`SaveLinkJob` should define the parameters that are required in its constructor, a.k.a the job's *signature*, rather than accessing
the data from the request so that we can call this job from other places in our application (e.g. from a command or a custom class)
and not be restricted by the protocol, in this case HTTP request.

We use this technique to increase the degree of job isolation and secure the single responsibility principle.

```php
<?php

namespace App\Domains\Link\Jobs;

use Lucid\Units\Job;
use App\Data\Models\Link;

class SaveLinkJob extends Job
{
    private $url;
    private $title;
    private $description;

    /**
     * Create a new job instance.
     *
     * @param $url
     * @param $title
     * @param $description
     */
    public function __construct($url, $title, $description)
    {
        $this->url = $url;
        $this->title = $title;
        $this->description = $description;
    }

    /**
     * Execute the job.
     *
     * @return Link
     */
    public function handle()
    {
        $attributes = [
            'url' => $this->url,
            'title' => $this->title,
            'description' => $this->description,
        ];

        return tap(new Link($attributes))->save();
    }
}
```

The job's signature is its constructor: `__construct($url, $title, $description)` telling us what's required to run this job.
This gets easier to read with PHP 7+ where we could type-hit these parameters:

```php
public function __construct(string $url, string $title, string $description)
```

And even better with PHP 8 we could use constructor property promotion and further reduce boilerplate:

```php
public function __construct(
    private string $url,
    private string $title,
    private string $description
) {}
```

Then we'll run this job from the feature to save links when received:

```php
<?php

namespace App\Features;

use Lucid\Units\Feature;
use App\Domains\Link\Requests\AddLink;
use App\Domains\Link\Jobs\SaveLinkJob;
use App\Domains\Http\Jobs\RedirectBackJob;

class AddLinkFeature extends Feature
{
    public function handle(AddLink $request)
    {
        $this->run(SaveLinkJob::class, [
            'url' => $request->input('url'),
            'title' => $request->input('title'),
            'description' => $request->input('description'),
        ]);
    }
}
```

Calling `$this->run($unit, $params)` in a feature causes the underlying dispatcher to run `SaveLinkJob` syncronously by calling its `handle` method
after initializing it with the provided `$params` which are passed as an associative array where the keys must match the job's
constructor parameters in naming, but not the order. So this would still work the same:

```php
$this->run(SaveLinkJob::class, [
    'description' => $request->input('description'),
    'title' => $request->input('title'),
    'url' => $request->input('url'),
]);
```

{{% notice info %}}
{{<icon name="fa-info-circle">}}&nbsp;You may call jobs (and other units) from any class by
supplying `Lucid\Bus\UnitDispatcher` trait in the class which will equip the `run($unit, $params)` function to run jobs and operations.
With this, the class in Lucid terms is now called a [custom dispatcher]({{<ref "/jobs#custom-dispatcher">}}).
{{% /notice %}}


The last step is to redirect a successful request back to the form. To do that we'll create a `RedirectBackJob`
which will simply call `back()`. Even though it might seem like an overhead, but remember that we're setting up for scale,
and as we scale, the less free-form code we have the better; instead of having plenty of `back()` and `back()->withInput()`
and other calls, we centralize them in a job so that in case we ever wanted to modify that functionality or add to it,
the change will only need to happen in a one place.

`RedirectBackJob` will reside in a new `Http` domain, a place for all our HTTP-related functionality that isn't specific to a
business entity of our application, fits the abstract type of domains instead of the entity type.

```bash
lucid make:job RedirectBackJob http
```

```bash
Job class RedirectBackJob created successfully.

Find it at app/Domains/Http/Jobs/RedirectBackJob.php
```

Our job will provide the option `withInput` to determine whether input should be included in the redirection.
This is a simple example of how such a simple job may later provide functionality that can be shared across the application.

```php
<?php

namespace App\Domains\Http\Jobs;

use Lucid\Units\Job;

class RedirectBackJob extends Job
{
    /**
     * @var bool
     */
    private $withInput;

    /**
     * Create a new job instance.
     *
     * @param bool $withInput
     */
    public function __construct($withInput = false)
    {
        $this->withInput = $withInput;
    }

    /**
     * Execute the job.
     */
    public function handle()
    {
        $back = back();

        if ($this->withInput) {
            $back->withInput();
        }

        return $back;
    }
}

```

## Testing

If you visit `/submit` fill the form it should now add the links, but to be certain about the functionality we just built
it is necessary that we write some tests to ensure it continues to.

### Configure PHPUnit

First we need to configure the database to run in a memory SQLite database instance, in `phpunit.xml`
uncomment the following lines:

```xml
<server name="DB_CONNECTION" value="sqlite"/>
<server name="DB_DATABASE" value=":memory:"/>
```

And for domain tests to be automatically detected by PHPunit,
we'll need to add the following snippet to `phpunit.xml` within the `<testsuites>` tag:

```xml
<testsuite name="Domains">
    <directory suffix="Test.php">./app/Domains</directory>
</testsuite>
<testsuite name="Features">
    <directory suffix="Test.php">./tests/Features</directory>
</testsuite>
```

{{% notice secondary %}}
This also allows us to run this suite in isolation with `phpunit --testsuite Domains`
{{% /notice %}}

### Unit Tests

Jobs in Lucid are units, and their tests are that of a unit test where we verify that all the variations of the data it might
receive wouldn't misbehave unexpectedly.

Let's write a test for `SaveLinkJob` in `app/Domains/Link/Tests/Jobs/SaveLinkJobTest` which has already been created by `lucid`
when generating the job:

{{% notice danger %}}
{{<icon name="fa-exclamation-triangle">}}&nbsp;Runnig tests prior to configuring `phpunit.xml` as mentioned above will wipe
out the data that is currently in your database.
{{% /notice %}}

```php
<?php

namespace App\Domains\Link\Tests\Jobs;

use Tests\TestCase;
use App\Data\Models\Link;
use Faker\Factory as Fake;
use App\Domains\Link\Jobs\SaveLinkJob;
use Illuminate\Foundation\Testing\RefreshDatabase;

class SaveLinkJobTest extends TestCase
{
    use RefreshDatabase;

    public function test_save_link_job()
    {
        $f = Fake::create();

        $url = $f->url;
        $title = $f->sentence;
        $description = $f->paragraph;

        $job = new SaveLinkJob($url, $title, $description);
        $link = $job->handle();

        $this->assertInstanceOf(Link::class, $link);
        $this->assertEquals($url, $link->url);
        $this->assertEquals($title, $link->title);
        $this->assertEquals($description, $link->description);
    }
}

```

For more on testing jobs visit [jobs#testing]({{<ref "/jobs#testing">}}).

### Feature Test

The last test is the feature's behaviour with different input variations and make sure that all responses are as expected.
In principle, Lucid's feature tests are about testing the integration between the units that the feature runs (jobs and operations).

Starting with our test layout as a plan to what we will be examining and prepare the test class by including `RefreshDatabase` trait:

`tests/Features/AddLinkFeatureTest`
```php
<?php

namespace App\Tests\Features;

use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

class AddLinkFeatureTest extends TestCase
{
    use RefreshDatabase;

    public function test_guest_cannot_submit_a_link()
    {
        $this->markTestIncomplete();
    }

    public function test_link_is_not_created_if_validation_fails()
    {
        $this->markTestIncomplete();
    }

    public function test_link_is_not_created_with_invalid_url()
    {
        $this->markTestIncomplete();
    }

    public function test_max_length_fails_when_too_long()
    {
        $this->markTestIncomplete();
    }

    public function test_max_length_succeeds_when_under_max()
    {
        $this->markTestIncomplete();
    }
}
```

Now we'll just fill the tests with corresponding calls and assertions:

We expect a guest to not be able to submit links since in our Request class `AddLink::authorize()` requires authorization:
```php
public function test_guest_cannot_submit_a_link()
{
    $response = $this->post('/submit', [
        'title' => 'Example Title',
        'url' => 'http://example.com',
        'description' => 'Example description.',
    ]);

   $response->assertStatus(403);
   $response->assertSee('This action is unauthorized.');
}
```

Ensure input validation is as expected:
```php
public function test_link_is_not_created_if_validation_fails()
{
    $response = $this->actingAs(User::factory()->create())->post('/submit');

    $response->assertSessionHasErrors(['title', 'url', 'description']);
}
```

The purpose of the data provider `invalidURLs` here is to keep the test concise and reduce clutter:
```php
public function invalidURLs()
{
    return [
        ['foo.com'],
        ['/invalid-url'],
        ['//invalid-url.com'],
    ];
}

/**
 * @dataProvider invalidURLs
 */
public function test_link_is_not_created_with_invalid_url($case)
{
    $response = $this->actingAs(User::factory()->create())
                     ->post('/submit', [
                        'url'  => $case,
                        'title' => 'Example Title',
                        'description' => 'Example description',
                    ]);

    $response->assertSessionHasErrors(['url' => 'The url format is invalid.']);
}
```

Finally, test input strings lengths:

```php
public function test_max_length_fails_when_too_long()
{
    $title = str_repeat('a', 256);
    $description = str_repeat('a', 256);
    $url = 'http://';
    $url .= str_repeat('a', 256 - strlen($url));

    $user = User::factory()->create();
    $response = $this->actingAs($user)
                     ->post('/submit', compact('title', 'url', 'description'));

    $response->assertSessionHasErrors([
        'url' => 'The url may not be greater than 255 characters.',
        'title' => 'The title may not be greater than 255 characters.',
        'description' => 'The description may not be greater than 255 characters.',
    ]);
}

public function test_max_length_succeeds_when_under_max()
{
    $url = 'http://';
    $url .= str_repeat('a', 255 - strlen($url));

    $data = [
        'title' => str_repeat('a', 255),
        'url' => $url,
        'description' => str_repeat('a', 255),
    ];

    $this->actingAs(User::factory()->create())->post('/submit', $data);

    $this->assertDatabaseHas('links', $data);
}
```

## Conclusion

As far as this example goes, Lucid may seem like just an overhead on top of simple one-liners or two.
Pragmatically speaking, this is almost never the case; no project begins as simple remains so simple in a few weeks later
and this is where this architecture's role becomes crucial. It is just like wine - gets better with the project's age.

### Highlights

Here are a few points that are worth mentioning to showcase where the architecture came in and how it would help forward:

- We are still using Laravel's internals and `artisan` for most of the things we did. This is intended to show how Lucid preserves
Laravel's defaults and avoids replicating or replacing them; in fact it complements them by elevating
their presence and fitting them within a defined structure.

- Navigating features couldn't get any easier, by looking at the `app/Features` directory you'd be able to have a summary of what
this application does at a glance.

- Visiting a feature's class would also provide an overview of the steps required with the least
details possible, yet details are available when wanting to dig deeper.
In addition to the presence of tests that mirror these classes which makes it easy to update functionality with reduced unintentional negative impact.

- Expanding on functionality such as `SaveArticleJob` makes it easy to achieve a high degree of reusable code.
Consider updating a link, we'd still use this job to save an existing link by supplying an optional `id` parameter and have the
job create or update accordingly.
