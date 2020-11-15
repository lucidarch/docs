---
title: "Micro"
date: 2020-11-15T11:20:52Z
draft: true
weight: 1
hide: ["header"]
---

In this guide we're going to build a link directory where we can register and save links of our own using Lucid Micro -
the default variant for single-purpose applications.

From the Lucid stack we'd be using
[Features]({{<ref "/features">}}), [Jobs]({{<ref "/jobs">}}), [Opertaions]({{<ref "/operations">}}), [Domains]({{<ref "/domains">}}), [Requests]({{<ref "/validation#/#form-request-validation">}}) and Data
to build the following features:

- Display a list of links.
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
    DB_DATABASE="storage/app/database/database.sql"
    ```

3. Test Configuration
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

We will be using a handful of moder technologies to demonstrate how they organically fit within the Lucid Architecture due to its
timeless approach towards application structure.

- [TailwindCSS](https://tailwindcss.com) for styling
- [Laravel Blade](https://laravel.com/docs/blade) for templating
- [Frontend Presets](https://github.com/laravel-frontend-presets/tailwindcss) for scaffolding

### Authentication

We will try to avoid re-inventing the wheel as much as possible by using a package that configures our project to use with TailwindCSS
and scaffold basic authentication views, controllers and routes.

```bash
composer require laravel-frontend-presets/tailwindcss --dev
```

``` bash
php artisan ui tailwindcss --auth
```

```bash
npm install && npm run dev
```

Now to watch assets and build on change we may run `npm run watch`.

The resulting files are under `resources/views/auth` and the `welcome` page has been updated to look as follows:

![Welcome](/media/images/getting-started/welcome.png)

To create an account click on Register at the top right and enter your account details to be logged in to the dashboard:

![Register](/media/images/getting-started/register.png)

### Link Submission

First, create a new route in `routes/web.php`:

```php
Route::get('/submit', function() {
    return view('submit');
});
```

Next, we need to create the `submit.blade.php` template at `resources/views/submit.blade.php` with the following boilerplate
to submit a link with a title and a description:

```html
@extends('layouts.app')
@section('content')
    <div class="min-h-screen flex items-center justify-center">
        <div class="flex flex-col justify-around h-full">
            <div>
                <form action="/submit" method="post" class="w-full max-w-sm bg-white shadow-md rounded px-8 pt-6 pb-8 mb-4">
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
        </div>
    </div>
@endsection

```

Generate `LinkController` to manage our links using `lucid` to have it ready for serving features.

```bash
lucid make:controller Link
```

```bash
Controller class created successfully.

Find it at app/Http/Controllers/LinkController.php
```

{{% notice secondary %}}
Notice the automatic addition of `Controller` suffix, used as a standard for naming conventions to match other suffixes such as `Feature`, `Job` and `Operation`.
{{% /notice %}}

We will need a method to handle the form's submittion request, we'll call it `add`:

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

What the `add` method will do is serve the feature that will run the jobs required to to add links.
Let's generate the feature `AddLinkFeature`:

```bash
lucid make:feature AddLink
```

```bash
Feature class AddLinkFeature created successfully.

Find it at app/Features/AddLinkFeature.php
```

`add()` will now serve `AddLinkFeature`

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

**Database & Model**

Before we can stat accepting data we need to prepare our database.
Starting with a migration to create the links table:

```bash
php artisan make:migration create_links_table --create=links
```

The generated file will contain the creation schema at `database/migrations/{datetime}_create_links_table`.
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

**Request Validation**

The first step of receiving input is to validate it. We will be using [Form Request validation](https://laravel.com/docs/validation#form-request-validation)
where each Request belongs in a Domain representing the entity that's being managed, in this case it's `Link` containing an `AddLink` Request class.

This will be the beginning of working with Domains in Lucid. They're used to organize Jobs and custom classes that contain logic related to a certain topic
according to the domain-driven design.

Starting with validation, Lucid positions Request classes within their corresponding domains. To generate a `AddLink` use:

```bash
lucid make:request AddLink link
```

```bash
Request class created successfully.

Find it at app/Domains/Link/Requests/AddLink.php
```

In the `AddLink` we'll need to update the methods `authorize()` and `rules()` to validate the input:

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
the request class in the feature's `handle()` method:

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

![Validation errors UI](/media/images/getting-started/validation-errors.png)



To save the link we'll create a job that saves links and run it in our feature, which will be added to our `Link` domain at
`app/Domains/Link/Jobs/SaveLinkJob` along with its test `app/Domains/Link/Tests/Jobs/SaveLinkJobTest`.

```bash
lucid make:job SaveLik link
```

{{% notice info %}}
{{<icon name="fa-asterisk">}}&nbsp;Notice the naming that we use with jobs, here we called it `SaveLinkJob`
so that we can reuse whenever needed by extending its functionality futher as we will see through this guide.
{{% /notice %}}

`SaveLinkJob` should define the parameters that are required in its constructor, acting as the job's *signature*, rather than accessing
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

And even better with PHP 8 we could use constructor property promotion:

```php
public function __construct(
    private string $url,
    private string $title,
    private string $description
) {}
```

Then we'll run this job from the feature:

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

As a last step we need to redirect a successful request back to the form. To do that we'll create a `RedirectBackJob`
which will simply call `back()`. Even though it might seem like an overhead, but remember that we're setting up for scale,
and as we scale, the less free-form code we have the better; instead of having plenty of `back()` and `back()->withInput()`
and other calls, we centralize them in a job so that in case we ever wanted to modify that functionality we'll be change in a single place.

`RedirectBackJob` will reside in a new `Http` domain, a place for all our HTTP-related functionality such as responses.

```bash
lucid make:job RedirectBackJob http
```

```bash
Job class RedirectBackJob created successfully.

Find it at app/Domains/Http/Jobs/RedirectBackJob.php
```

Our class will provide the option `withInput` to determine whether input should be included in the redirection.
A simple example of how such a job may later provide functionality that can be shared across the application.

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

## Outcomes
- We are still using Laravel's artisan for most of the commands that we are running. This is intended to showcase how Lucid preserves
Laravel's defaults and avoids replicating them; only takes care of elevating their presence and fitting it within a defined structure.
