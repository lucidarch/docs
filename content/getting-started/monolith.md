---
title: "Monolith"
date: 2020-11-15T11:20:42Z
draft: false
weight: 2
hide: ["header"]
---

{{% notice light %}}
<i class="fab fa-github fa-lg"></i>&nbsp;The source code for this exercise is on [GitHub](https://github.com/lucidarch/getting-started-monolith).
{{% /notice %}}

In this guide we're going to build an application to share and discuss food recipes that's made up of two sections:

1. **Kitchen**: To manage and share recipes, calculating calories and avg. price.
2. **Forum**: To discuss recipes with fellow members and get in touch with the experts.

The purpose of this exercise is to demonstrate Lucid Monolith - the multi-purpose, service-oriented variant of Lucid.
Typically used when you wish to apply separation of concerns yet still be able to share code that is necessary for all
the parts such as models and business logic.

The obvious use of Monolith is to contain a large feature set within a single codebase, in contrast with Microservices
where we would dissect it into several codebases and have them responsible for their own part in isolation yet integrate
and communicate to get a single job done. Starting with Lucid's Monolith grants the ability to later do the transformation, making
it easier to decide on the different parts that need to go into their own [micro]services, and the code would've already been
concentrated to minimise the move on their own.

{{% notice secondary %}}
{{<icon name="fa-archway">}}&nbsp;For Micro - the single-purpose variant, see [Getting Started • Micro]({{<ref "/getting-started/micro">}})
{{% /notice %}}

## Setup

### Install Laravel

Let's start by creating a new Laravel project. It is best if you refer to [Laravel's installation docs](https://laravel.com/docs/installation)
and choose your preferred way of installation, but here are the common ways to do it:

```bash
# via the installer
laravel new cookery

# via composer
composer create-project --prefer-dist laravel/laravel cookery
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

## Initialize Monolith

Assuming that you have `./vendor/bin` in your `PATH` the `lucid` binary will be available to run [Lucid commands]({{<ref "/cli">}}).

We will start with `init:monolith` to initialize directory structure and specify our first service to be created along.

```bash
lucid init:monolith Kitchen
```

```yaml
Initializing Lucid Monolith for Laravel 8.15.0

Created directories:
/app/Data
/app/Domains
/app/Services
/app/Foundation
/app/Policies
Service Kitchen created successfully.

Activate it by adding App\Services\Kitchen\Providers\KitchenServiceProvider::class
to 'providers' in config/app.php
```

Let's do as instructed and add the service's provider to the `providers` array in `config/app.php` and this way Laravel will recognize our Kitchen
service and load its files.

Now if we run our application with `php artisan serve` and visit `/kitchen` we should be greeted by the service.
By default the service's routes are prefixed with its name. Of course, you may modify that at `app/Services/Kitchen/routes`
which has the exact same structure as Laravel's default `/routes`.

## Service Directory Structure

Below is the default directory structure of every service in the monolith. They look familiar because they are the same as Laravel's
default structure, intended to provide the same functionality but with a different arrangement.

The highlighted directories are those that are proprietary to Lucid as we will learn throughout this guide.

<pre>
app/Services/Kitchen
├── Console
│   └── Commands
├── <strong>Features</strong>
├── <strong>Operations</strong>
├── Http
│   ├── Controllers
│   └── Middleware
├── Providers
│   ├── ApiServiceProvider.php
│   ├── BroadcastServiceProvider.php
│   └── RouteServiceProvider.php
├── <strong>Tests</strong>
│   └── <strong>Features</strong>
│   └── <strong>Operations</strong>
├── database
│   ├── factories
│   ├── migrations
│   └── seeds
├── resources
│   ├── lang
│   └── views
└── routes
    ├── api.php
    ├── channels.php
    ├── console.php
    └── web.php
</pre>

### The Service Provider

`KitchenServiceProvider` is where the service tells Laravel how to locate its files such as routes, views, lang
and it may be used for anything that the service may need in the future. Having this file allows us to control
the loading of serivce files conditionally.

The service files will not be loaded by Laravel if it weren't registered. We used the simple method of adding the provider
to the app's `providers` array which means that the service files will be registered in every interaction with our application.
This may not always be the intented behaviour, where we would want to register services conditionally.

For example, if we wish to load only Kitchen service if our domain matches `kitchen.example.com` we would instead register
the provider in `app/Providers/AppServiceProvider::regsiter` as such:

```php
public function regsiter()
{
    switch (Route::input('subdomain')) {
        case 'kitchen':
            $this->register(KitchenServiceProvider::class);
            break;
        case 'forum':
            $this->register(ForumServiceProvider::class);
            break;
    }
}
```

Given that we define our routes with a subdomain group:

`app/Services/Kitchen/routes/web.php`
```php
Route::group(['domain' => '{subdomain}.cookery.local'], function() {

    // define Kitchen routes here

});
```

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

{{% notice info %}}
Authentication is central to our application, the same user will be accessing both the forum and the kitchen.
We're keeping auth and `User` at root for the brevity of this example but ideally we'd move the `User` model to our `Data/Models` directory
and adapt Auth to read it from there in `config/auth.php`, though this is not necessary for now.
{{% /notice %}}

## Recipe Submission

The first feature we will work on is to receive a recipe's details and add it to our records. Starting with the form:

### View

`app/Services/Kitchen/routes/web.php`
```php
Route::group(['prefix' => 'kitchen'], function() {

    Route::get('/recipes/new', function() {
        return view('kitchen::new');
    });

});

```

Next, we need to create the form at `app/Services/Kitchen/resources/views/new.blade.php` with the styles:

`resources/css/app.css`
```scss
@import 'tailwindcss/base';
@import 'tailwindcss/components';
@import 'tailwindcss/utilities';

@layer components {
    .form-container {
        @apply flex items-center justify-center h-screen;
    }

    .form {
        @apply w-full max-w-sm bg-white shadow-md rounded px-8 pt-6 pb-8;
    }

    .form-label {
        @apply block text-gray-500 font-bold mb-1 pr-4;
    }

    .form-error-status-message {
        @apply bg-red-100 border border-red-400 text-red-700 px-4 py-3 mt-2 mb-2 rounded relative;
    }

    .form-input-row {
        @apply flex items-center mb-6;
    }

    .form-input-error-label {
        @apply text-red-500 text-xs italic mt-2;
    }

    .form-input {
        @apply appearance-none border-2 rounded w-full py-2 px-4 text-gray-700 leading-tight border-gray-200;
    }

    .form-input:focus {
        @apply bg-white border-teal-500 outline-none;
    }

    .btn {
        @apply shadow bg-teal-500 text-white font-bold py-2 px-4 rounded;
    }

    .btn:hover {
        @apply bg-teal-400
    }

    .btn:focus {
        @apply shadow-outline outline-none;
    }
}

```

{{% notice info %}}
It is recommended to scope styles per service as well, but it would've required a couple more steps that we'd rather
skip for being out of the scope of this guide. i.e. loading a custom CSS file from `app/Services/Kitchen/resources/css/kitchen.css`
{{% /notice %}}

`app/Services/Kitchen/resources/views/new.blade.php`
```html
<x-app-layout>
    <x-slot name="header">
        Add Recipe
    </x-slot>

    <div class="form-container">
        <form action="/submit" method="post" class="form">
            @csrf
            @if ($errors->any())
                <div class="form-error-status-message" role="alert">
                    Please fix the following errors
                </div>
            @endif
            <div class="form-input-row">
                <div class="md:w-1/3">
                    <label class="form-label" for="title">
                        Title
                    </label>
                </div>
                <div class="md:w-2/3">
                    <input id="title"
                           type="text"
                           name="title"
                           value="{{ old('title') }}"
                           class="form-input">
                    @error('title')
                    <p class="form-input-error-label">{{ $message }}</p>
                    @enderror
                </div>
            </div>

            <div class="form-input-row">
                <div class="md:w-1/3">
                    <label for="ingredients" class="form-label" for="desription">
                        Ingredients
                    </label>
                </div>
                <div class="md:w-2/3">
                    <p class="text-gray-500 text-xs">each ingredient on a new line.</p>
                    <p class="text-gray-500 text-xs">Ingredient, mass /g, $ /g</p>
                    <textarea id="ingredients"
                              name="ingredients"
                              class="form-input h-48"
                              placeholder="Avocado, 0.5, 0.07
                              Lettuce, 0.3, 0.04">{{ old('ingredients') }}</textarea>
                    @error('ingredients')
                    <p class="form-input-error-label">{{ $message }}</p>
                    @enderror
                </div>
            </div>

            <div class="form-input-row">
                <div class="md:w-1/3">
                    <label for="instructions" class="form-label" for="desription">
                        Instructions
                    </label>
                </div>
                <div class="md:w-2/3">
                    <textarea id="instructions"
                              name="instructions"
                              class="form-input h-24"
                              placeholder="How to do it?">{{ old('instructions') }}</textarea>
                </div>
            </div>

            <div class="md:flex md:items-center">
                <div class="md:w-1/3"></div>
                <div class="md:w-2/3">
                    <button type="submit" class="btn">
                        Add
                    </button>
                </div>
            </div>
        </form>
    </div>
</x-app-layout>
```

It's a simple form:

{{% panel %}}
![Add Recipe Form](/media/images/getting-started/add-recipe.png)
{{% /panel %}}

### Controller

Generate `RecipeController` in Kitchen service to handle recipe requests and serve corresponding features:

```bash
lucid make:controller recipe kitchen
```

```
Controller class created successfully.

Find it at app/Services/Kitchen/Http/Controllers/RecipeController.php
```

{{% notice info %}}
Notice the automatic addition of `Controller` suffix, used as a naming convention to match other suffixes in the Lucid stack such as `Feature`, `Job` and `Operation`.
{{% /notice %}}

We will need a method to handle the form's submission request, let's call it `add`:

`app/Services/Kitchen/Http/Controllers/RecipeController.php`

```php
<?php

namespace App\Services\Kitchen\Http\Controllers;

use Lucid\Units\Controller;

class RecipeController extends Controller
{
    public function add()
    {

    }
}
```

### Feature

The `add` method will then serve the feature that will run the jobs and operations required to add recipes.
Generate a feature called `AddRecipeFeature` in Kitchen service:

```bash
lucid make:feature AddRecipe kitchen
```

```bash
Feature class AddRecipeFeature created successfully.

Find it at app/Services/Kitchen/Features/AddRecipeFeature.php
```

`add()` will now serve `AddRecipeFeature`

`app/Services/Kitchen/Http/Controllers/RecipeController.php`

```php
use App\Services\Kitchen\Features\AddRecipeFeature;

...

public function add()
{
    return $this->serve(AddRecipeFeature::class);
}
```

Beginning with the step of generating a feature will cognitively set our context to what we will be working on, helping us concentrate
on the task at hand.

Let's expose our feature with a route:

```php
use App\Services\Kitchen\Http\Controllers\RecipeController;

Route::group(['prefix' => 'kitchen'], function() {

    Route::post('/recipes', [RecipeController::class, 'add']);

});

```

### Database

Before we can start accepting data we need to prepare our database with a migration to create the recipes table:

We would like our kitchen tables to be managed by the Kitchen service, so we will place the migrations
in `app/Services/Kitchen/database/migraitons` instead of the default `database/migrations`.
This is an optional step and may not fit all cases, but the option is here when needed using the `lucid make:migration`:

```bash
lucid make:migration create_recipes_table kitchen
```

```
Created Migration: 2020_11_18_211503_create_recipes_table


Find it at app/Services/Kitchen/database/migrations
```

Now we update the table's schema at `app/Services/Kitchen/database/migrations/{datetime}_create_recipes_table.php`
to include recipe fields and a reference to the user:

```php
Schema::create('recipes', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('ingredients');
    $table->text('instructions')->nullable();
    $table->decimal('price', 8, 3);
    $table->unsignedBigInteger('user_id');
    $table->timestamps();

    $table->foreign('user_id')
          ->references('id')
          ->on('users');
});
```

The migration will automatically be registered so the next time we run `php artisan migrate` `recipes` table will be created:

```bash
php artisan migrate
```

```
Migrating: 2020_11_18_211503_create_recipes_table
Migrated:  2020_11_18_211503_create_recipes_table (5.43ms)
```

### Model

{{% panel %}}
**The Data Directory `app/Data`**

For a scalable set of interconnected data elements, with Lucid we place them in `app/Data`,
because most likely over time writing the application there could develop a need for more than Models in data,
such as Repositories, Value Objects, Collections and more... which all fit in a central directory to consolidate them.
{{% /panel %}}

Create the `Recipe` model class at `app/Data/Models/Recipe.php` with the fields in `$fillable` for mass assignment:

```php
<?php

namespace App\Data\Models;

use Illuminate\Database\Eloquent\Model;

class Recipe extends Model
{
    protected $fillable = ['title', 'ingredients', 'instructions', 'price', 'user_id'];
}
```

### Request Validation

The first step of receiving input is to validate it. We will be using [Form Request validation](https://laravel.com/docs/validation#form-request-validation)
where each Request belongs in a Domain representing the entity that's being managed, in this case it's `Recipe` containing an `AddRecipe` Request class.

This will be the beginning of working with Domains in Lucid. They're used to group Jobs and custom classes which logic is
associated with certain topic according to domain-driven design.

Starting with validation, Lucid places Request classes within their corresponding domains. Let's generate an `AddRecipe` request:

{{% notice light %}}
{{<icon name="fa-asterisk">}}&nbsp;About the naming of the request class, it also would be an option to name it `CreateRecipe`
instead of `AddRecipe` to fulfill the CRUD convention, but `AddRecipe` was easier to read and pronounce in this particular case.
{{% /notice %}}

```bash
lucid make:request AddRecipe recipe
```

```bash
Request class created successfully.

Find it at app/Domains/Recipe/Requests/AddRecipe.php
```

In `AddRecipe` we'll need to update the methods `authorize()` and `rules()` to validate the request and its input.
Validation rules are the essentials:

```php
<?php

namespace App\Domains\Recipe\Requests;

use Illuminate\Support\Facades\Auth;
use Illuminate\Foundation\Http\FormRequest;

class AddRecipe extends FormRequest
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
            'ingredients' => ['required', 'max:255'],
            'instructions' => ['max:255'],
        ];
    }
}
```


With our request ready, now we need `AddRecipeFeature` to use that request class when served. We can do that by simply injecting
the request class in the feature's `handle()` method and every time the feature is served this validation will be applied.

```php
<?php

namespace App\Services\Kitchen\Features;

use Lucid\Units\Feature;
use App\Domains\Recipe\Requests\AddRecipe;

class AddRecipeFeature extends Feature
{
    public function handle(AddRecipe $request)
    {

    }
}

```

Now if we visit `/submit` and click `Add` wihtout passing any input it will generate errors and print their messages from validation failures.

![Validation errors UI](/media/images/getting-started/recipes-validation-errors.png)

### Calculate Recipe Price

To calculate the recipe price we need to do the following:

1. Parse the ingredients into an associative array with: `name`, `quantity` and `price`
2. Get each ingredient's usage price: `quantity x price`
3. Sum all ingredient usage prices to get recipe price `∑ ingredient price`

#### Value Objects

Looking at this requirement it seems that each step fits within a job, but thinking a little longer spawns the idea of Value Objects into thought.
Especially that there will be data exchange between several jobs.

The prominent idea would be to have an `Ingredient` object that's passed around instead of a primitive data type - associative array.
It will make it easier for someone reading the code to tell the structure (fields) by referring to the class rather than having to guess
and run after associative array key assignment instructions to figure out the final structure.

In other words, let's represent our `Ingredient` as a Value Object! Unlike models, value objects are only to be used at run
time and never stored.

{{% notice info %}}
{{<icon name="fa-exclamation">}}&nbsp;This concept is NOT related to the Lucid architecture although
endorsed for scale. For more on value objects, Martin Fowler put it best in [ValueObject](https://martinfowler.com/bliki/ValueObject.html)
in the domain-driven design section.
{{% /notice %}}

This brings up another advantage for the `app/Data` directory which will hold our value objects in `app/Data/Values` next to `app/Data/Models`.

Create `Ingredient` value object at `app/Data/Values/Ingredient.php` and we make it optional to have `quantity` and `price`

```php
<?php

namespace App\Data\Values;

class Ingredient
{
    public string $name;
    public float $quantity;
    public float $price;

    public function __construct(string $name, float $quantity = 0, float $price = 0)
    {
        $this->name = $name;
        $this->quantity = $quantity;
        $this->price = $price;
    }

}

```

#### Step 1: Parse Ingredients

Now we create a job that receives the `Recipe` model and parses `ingredient` field into `Ingredent` instances, calling it
`ParseIngredientsJob`.

Since we will be dealing with several instances, `Collection` classes are a powerful superset to array and it woule be perfect
to extend and make it easier for the receiver of the results to use and extract value from.

Let's create a specific collection for ingredients to pass around in `app/Data/Collections/IngredientsCollection.php`

```php
<?php

namespace App\Data\Collections;

use Illuminate\Support\Collection;

/**
 * A collection of Ingredient instances.
 */
class IngredientsCollection extends Collection
{

}
```

Now we can type-hint for it and visually represent what we're expecting in this collection.

Next is `ParseIngredientsJob` that will have the logic for this transformation from a string of ingredients
to `IngredientsCollection` containing `Ingredient` value object instances.

```bash
lucid make:job ParseIngredientsJob recipe
```

```bash
Job class ParseIngredientsJob created successfully.

Find it at app/Domains/Recipe/Jobs/ParseIngredientsJob.php
```

```php
<?php

namespace App\Domains\Recipe\Jobs;

use Lucid\Units\Job;
use App\Data\Models\Recipe;
use App\Data\Values\Ingredient;
use App\Data\Collections\IngredientsCollection;

class ParseIngredientsJob extends Job
{
    private string $ingredients;

    /**
     * Create a new job instance.
     *
     * @param string $ingredients
     */
    public function __construct(string $ingredients)
    {
        $this->ingredients = $ingredients;
    }

    /**
     * Execute the job.
     *
     * @return IngredientsCollection
     */
    public function handle(): IngredientsCollection
    {
        $ingredients = new IngredientsCollection();

        foreach (array_filter(explode("\r\n", $this->ingredients)) as $line) {
            $ingredient = new Ingredient(...explode(',', $line));
            $ingredients->push($ingredient);
        }

        return $ingredients;
    }
}
```

We have oursleves a job that can be called anywhere we receive a string of ingredients and would like to parse it.

A bit about the code above:

- `array_filter(explode("\r\n", $this->ingredients))` turns new lines into array elements and removes empty ones
- `new Ingredient(...explode(',', $line))` creates a new `Ingredients` instance by turning comma separated string into parameters

The returned result from that job would look like this:

```php
App\Data\Collections\IngredientsCollection {
  items: [
    0 => App\Data\Values\Ingredient {
      name: "Avocado"
      quantity: 0.5
      price: 0.07
      total: 0.035
    }
    1 => App\Data\Values\Ingredient {
      name: "Lettuce"
      quantity: 0.3
      price: 0.04
      total: 0.012
    }
  ]
```

#### Step 2: Calculate Ingredient's Total Price

To calculate an ingredient's own total price we simply do `$quantity * $price` but the question is where should this code be?

The thinking process to figure that out starts with the question "where would the next person look for this?" and the best answer to it wins.
In this case, to know the price of an ingredient, it is obvious that the first place to look is the `Ingredient` value object.
Becuase if we were to put it somewhere else outside the class, the next time we visit `Ingredient` we'd see that it recieves the total
price as an argument in its constructor but then we'd have to search for the place where this is being set.
Besides having an hidden dependency between classes.

Let's update `Ingredient` class to calculate its own total:

```php
<?php

namespace App\Data\Values;

class Ingredient
{
    public string $name;
    public float $quantity;
    public float $price;
    public float $total;

    public function __construct(string $name, float $quantity = 0, float $price = 0)
    {
        $this->name = $name;
        $this->quantity = $quantity;
        $this->price = $price;
        $this->total = $this->calculateTotal();
    }

    public function calculateTotal()
    {
        return $this->quantity * $this->price;
    }
}
```

#### Step 3: Calculate Recipe Total Price

Let's create a job to calculate the total price from `IngredientsCollection` calling it `CalculateIngredientsTotalJob` and returns
a single `float` value.

```bash
lucid make:job CalculateIngredientsTotalJob recipe
```

```bash
Job class CalculateIngredientsTotalJob created successfully.

Find it at app/Domains/Recipe/Jobs/CalculateIngredientsTotalJob.php
```

```php
<?php

namespace App\Domains\Recipe\Jobs;

use Lucid\Units\Job;
use App\Data\Collections\IngredientsCollection;

class CalculateIngredientsTotalJob extends Job
{
    private IngredientsCollection $ingredients;

    /**
     * Create a new job instance.
     *
     * @param IngredientsCollection $ingredients
     */
    public function __construct(IngredientsCollection $ingredients)
    {
        $this->ingredients = $ingredients;
    }

    /**
     * Execute the job.
     *
     * @return float
     */
    public function handle(): float
    {
        return $this->ingredients->sum('total');
    }
}
```

#### Run The Jobs

Now in `AddRecipeFeature` we run the jobs:

```php
public function handle(AddRecipe $request)
{
    $ingredients = $this->run(ParseIngredientsJob::class, [
        'ingredients' => $request->input('ingredients'),
    ]);

    $price = $this->run(new CalculateIngredientsTotalJob($ingredients));
}
```

#### Final Step: Group Jobs In Operations

It is indispensable for `ParseIngredientsJob` and `CalculateIngredientsTotalJob` to run together every time we need to calculate a recipe's price.
This case spawns [Operations]({{<ref "/operations">}}) into thought, to group these steps in one operation
that we can simply run with Lucid `$this->run(new CalculateRecipePriceOperation($ingredient))` which runs jobs performing both steps above.
Moreover, we would still be able to run any of the jobs separatly in different conditions if needed.

```bash
lucid make:operation CalculateRecipePrice kitchen
```

```bash
Operation class CalculateRecipePriceOperation created successfully.

Find it at app/Services/Kitchen/Operations/CalculateRecipePriceOperation.php
```

```php
<?php

namespace App\Services\Kitchen\Operations;

use Lucid\Units\Operation;
use App\Data\Collections\IngredientsCollection;
use App\Domains\Recipe\Jobs\ParseIngredientsJob;
use App\Domains\Recipe\Jobs\CalculateIngredientsTotalJob;

class CalculateRecipePriceOperation extends Operation
{
    private string $ingredients;

    public function __construct(string $ingredients)
    {
        $this->ingredients = $ingredients;
    }

    public function handle(): float
    {
        $ingredients = $this->run(ParseIngredientsJob::class, [
            'ingredients' => $this->ingredients,
        ]);

        return $this->run(new CalculateIngredientsTotalJob($ingredients));
    }
}
```

Then in the feature we'd replace the two jobs with just one call:

```php
public function handle(AddRecipe $request)
{
    $price = $this->run(CalculateRecipePriceOperation::class, [
        'ingredients' => $request->input('ingredients'),
    ]);
}
```

### Save Recipes

To save the recipe we'll create a job that saves recipes and run it in our feature, which will be added to our `Recipe` domain at
`app/Domains/Recipe/Jobs/SaveRecipeJob` along with its test `app/Domains/Recipes/Tests/Jobs/SaveRecipeJobTest`.

```bash
lucid make:job SaveRecipe recipe
```

```bash
Job class SaveRecipeJob created successfully.

Find it at app/Domains/Recipe/Jobs/SaveRecipeJob.php
```

{{% notice info %}}
{{<icon name="fa-asterisk">}}&nbsp;Notice the naming that we've used with this job "`SaveRecipeJob`" in contrast with "`AddRecipe`".
It is intended for reuse whenever needed by extending its functionality futher, for example `UpdateRecipe` feature may be able to use the same job.
{{% /notice %}}

`SaveRecipeJob` should define the parameters that are required in its constructor, a.k.a the job's *signature*, rather than accessing
the data from the request so that we can call this job from other places in our application (e.g. from a command or a custom class)
and not be restricted by the protocol, in this case HTTP request.

We use this technique to increase the degree of job isolation and secure the single responsibility principle.

```php
<?php

namespace App\Domains\Recipe\Jobs;

use Lucid\Units\Job;
use App\Models\User;
use App\Data\Models\Recipe;

class SaveRecipeJob extends Job
{
    private string $title;
    private string $ingredients;
    private string $instructions;
    private string $price;
    private User $user;

    /**
     * Create a new job instance.
     *
     * @param $title
     * @param $ingredients
     * @param $instructions
     * @param $price
     * @param User $user
     */
    public function __construct($title, $ingredients, $instructions, $price, User $user)
    {
        $this->title = $title;
        $this->ingredients = $ingredients;
        $this->instructions = $instructions;
        $this->price = $price;
        $this->user = $user;
    }

    /**
     * Execute the job.
     *
     * @return Recipe
     */
    public function handle(): Recipe
    {
        $attributes = [
            'title' => $this->title,
            'ingredients' => $this->ingredients,
            'instructions' => $this->instructions,
            'price' => $this->price,
            'user_id' => $this->user->getKey(),
        ];

        return tap(new Recipe($attributes))->save();
    }
}

```

The job's signature is its constructor: `__construct($title, $ingredients, $instructions, $price)` telling us what's required to run this job.
This gets easier to read and use with PHP 7+ where we could type-hit these parameters:

```php
public function __construct(
    string $title,
    string $ingredients,
    string $instructions,
    int $price)
```

And even better with PHP 8 we could use constructor property promotion and further reduce boilerplate:

```php
public function __construct(
    private string $title,
    private string $ingredients,
    private string $instructions,
    private int $price
) {}
```

{{% notice info %}}
{{<icon name="fa-lightbulb">}}&nbsp;**Note On Storage Approach**

We are taking a rather untraditional approach in this example. Easy on storage by saving ingredients as plain text rather than splitting them up into
their own model and building many-to-many relationships between recipes and ingredients, which may have been a better approach from a storage perspective.
However, for the sake of demonstrating a more complex case where we split them up and represent them in value objects,
because the relational approach is a familiar one and this example covers a wider range for variety reasons.
{{% /notice %}}

Then we'll run this job from the feature to save recipes when served:

```php
public function handle(AddRecipe $request)
{
    $price = $this->run(CalculateRecipePriceOperation::class, [
        'ingredients' => $request->input('ingredients'),
    ]);

    $this->run(SaveRecipeJob::class, [
        'title' => $request->input('title'),
        'ingredients' => $request->input('ingredients'),
        'instructions' => $request->input('instructions'),
        'price' => $price,
        'user' => Auth::user(),
    ]);
}
```

Calling `$this->run($unit, $params)` in a feature triggers the underlying dispatcher to run `SaveRecipeJob` instantly and syncronously by calling its `handle` method
after initializing it with the provided `$params` which are passed as an associative array where the keys must match the job's
constructor parameters in naming, but not the order. So this would still work the same:

```php
$this->run(SaveRecipeJob::class, [
    'instructions' => $request->input('instructions'),
    'ingredients' => $request->input('ingredients'),
    'price' => $price,
    'title' => $request->input('title'),
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

Finally, conclude the feature by responding:

```php
public function handle(AddRecipe $request)
{
    $price = $this->run(CalculateRecipePriceOperation::class, [
        'ingredients' => $request->input('ingredients'),
    ]);

    $this->run(SaveRecipeJob::class, [
        'title' => $request->input('title'),
        'ingredients' => $request->input('ingredients'),
        'instructions' => $request->input('instructions'),
        'price' => $price,
        'user' => Auth::user(),
    ]);

    return $this->run(RedirectBackJob::class);
}
```

## Testing

If you visit `/kitchen/recipes/new` and fill the form it should now add the recipes, but to be certain about the functionality we just built
it is necessary that we write some tests to ensure it continues to.

### Configure PHPUnit

First we need to configure the database to run in a memory SQLite database instance, in `phpunit.xml`
uncomment the following lines:

```xml
<server name="DB_CONNECTION" value="sqlite"/>
<server name="DB_DATABASE" value=":memory:"/>
```

And for domains and services tests to be automatically detected by PHPunit,
we'll need to add the following snippet to `phpunit.xml` within the `<testsuites>` tag:

```xml
<testsuite name="Domains">
    <directory suffix="Test.php">./app/Domains</directory>
</testsuite>
<testsuite name="Services">
    <directory suffix="Test.php">./app/Services/*/Tests</directory>
</testsuite>
```

{{% notice secondary %}}
This also allows us to run this suite in isolation with `phpunit --testsuite Domains`
{{% /notice %}}


### Unit Tests

Jobs in Lucid are units, and their tests are that of a unit test where we verify that all the variations of the data it might
receive wouldn't misbehave unexpectedly.

{{% notice danger %}}
{{<icon name="fa-exclamation-triangle">}}&nbsp;Runnig tests prior to configuring `phpunit.xml` as mentioned above will wipe
out the data that is currently in your database.
{{% /notice %}}

---

#### `ParseIngredientsJobTest`

Let's write a test for `ParseIngredientsJob` to make sure that it's failsafe, in `app/Domains/Recipe/Tests/Jobs/ParseIngredientsJobTest`
which has already been created by `lucid` when generating the job. The test should cover three conditions:

1. successful parsing
2. parsing empty string of ingredients
3. failsafe parsing with missing values

```php
<?php

namespace App\Domains\Recipe\Tests\Jobs;

use Tests\TestCase;
use App\Data\Collections\IngredientsCollection;
use App\Domains\Recipe\Jobs\ParseIngredientsJob;

class ParseIngredientsJobTest extends TestCase
{
    public function test_parse_ingredients_job()
    {
        $input = "Avocado, 1, 1.2\r\nLettuce, 0.4, 0.8";

        $job = new ParseIngredientsJob($input);

        $ingredients = $job->handle();

        $this->assertInstanceOf(IngredientsCollection::class, $ingredients);

        $this->assertEquals('Avocado', $ingredients[0]->name);
        $this->assertEquals(1.0, $ingredients[0]->quantity);
        $this->assertEquals(1.2, $ingredients[0]->price);
        $this->assertEquals(1.2, $ingredients[0]->total);

        $this->assertEquals('Lettuce', $ingredients[1]->name);
        $this->assertEquals(0.4, $ingredients[1]->quantity);
        $this->assertEquals(0.8, $ingredients[1]->price);
        $this->assertEquals(0.32, $ingredients[1]->total);
    }

    public function test_parsing_empty_ingredients()
    {
        $job = new ParseIngredientsJob("");

        $ingredients = $job->handle();

        $this->assertInstanceOf(IngredientsCollection::class, $ingredients);
        $this->assertTrue($ingredients->isEmpty());
    }

    public function test_failsafe_parsing_ingredients_with_missing_values()
    {
        $input = "Missing Price, 1\r\nOnly Title\r\n";

        $job = new ParseIngredientsJob($input);

        $ingredients = $job->handle();

        $this->assertInstanceOf(IngredientsCollection::class, $ingredients);
        $this->assertEquals(2, $ingredients->count());

        $this->assertEquals('Missing Price', $ingredients[0]->name);
        $this->assertEquals(1.0, $ingredients[0]->quantity);
        $this->assertEquals(0, $ingredients[0]->price);
        $this->assertEquals(0, $ingredients[0]->total);

        $this->assertEquals('Only Title', $ingredients[1]->name);
        $this->assertEquals(0, $ingredients[1]->quantity);
        $this->assertEquals(0, $ingredients[1]->price);
        $this->assertEquals(0, $ingredients[1]->total);
    }
}
```

{{% notice secondary %}}
{{<icon name="fa-exclamation-triangle">}}&nbsp;There is only one case where parsing fails with this implementation:
(i.e. `Avocado, ,2.5`) it is when skipping values. Consider it an exercise to improve the code! It was left out in this example to not complicate logic further.
{{% /notice %}}

For more on testing jobs visit [jobs#testing]({{<ref "/jobs#testing">}}).

The tests for the rest of the jobs are available in the [source code](https://github.com/lucidarch/getting-started-monolith).

---

#### `CalculateRecipeOperationTest`

Testing operations is fairly similar and also fits among unit testing. We'll right the tests in the generated file
`app/Services/Kitchen/Tests/Operations/CalculateRecipeOperationTest`

```php
<?php

namespace App\Services\Kitchen\Tests\Operations;

use Tests\TestCase;
use App\Services\Kitchen\Operations\CalculateRecipePriceOperation;

class CalculateRecipeOperationTest extends TestCase
{
    public function test_calculate_recipe_operation()
    {
        $input = "Avocado, 1, 1.2\r\nLettuce, 0.4, 0.8";

        $op = new CalculateRecipePriceOperation($input);

        $this->assertEquals(1.52, $op->handle());
    }

    public function test_calculating_empty_recipe_operation()
    {
        $op = new CalculateRecipePriceOperation("");

        $this->assertEquals(0.0, $op->handle());
    }

    public function test_calculating_recipe_with_missing_values_operation()
    {
        $input = "Missing Price, 1\r\nOnly Title\r\n";

        $op = new CalculateRecipePriceOperation($input);

        $this->assertEquals(0.0, $op->handle());
    }
}
```

For more on testing operations visit [operations#testing]({{<ref "/operations#testing">}})


### Feature Test

The last test if the feature's behaviour with different input variations and make sure that all responses are as expected.
In principle, Lucid's feature tests are about testing the integration between the units that the feature runs (jobs and operations).

Starting with our test layout as a plan to what we will be examining and prepare the test class by including `RefreshDatabase` trait:

`app/Services/Kitchen/Tests/Features/AddRecipeFeatureTest.php`

```php
<?php

namespace App\Services\Kitchen\Tests\Features;

use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;

class AddRecipeFeatureTest extends TestCase
{
    use RefreshDatabase;

    public function test_guest_cannot_submit_a_recipe()
    {
        $this->markTestIncomplete();
    }

    public function test_recipe_is_not_created_if_validation_fails()
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

We expect a guest to not be able to submit recipes since in our Request class `AddRecipe::authorize()` requires authorization:

```php
public function test_guest_cannot_submit_a_recipe()
{
    $response = $this->post('/kitchen/recipes', [
        'title' => 'Avocado Salad Starter',
        'ingredients' => "Avocado, 1, 1.2\r\nLettuce, 0.4, 0.8",
        'instructions' => 'Mix it with oil and enjoy!',
    ]);

    $response->assertStatus(403);
    $response->assertSee('This action is unauthorized.');
}
```

Ensure input validation is as expected:

```php
public function test_recipe_is_not_created_if_validation_fails()
{
    $response = $this->actingAs(User::factory()->create())->post('/kitchen/recipes');

    $response->assertSessionHasErrors(['title', 'ingredients']);
}
```

Finally, test input strings lengths:

```php
public function test_max_length_fails_when_too_long()
{
    $title = str_repeat('a', 256);
    $ingredients = str_repeat('a', 256);
    $instructions = str_repeat('a', 256);

    $user = User::factory()->create();
    $response = $this->actingAs($user)
        ->post('/kitchen/recipes', compact('title', 'ingredients', 'instructions'));

    $response->assertSessionHasErrors([
        'title' => 'The title may not be greater than 255 characters.',
        'ingredients' => 'The ingredients may not be greater than 255 characters.',
        'instructions' => 'The instructions may not be greater than 255 characters.',
    ]);
}

public function test_max_length_succeeds_when_under_max()
{
    $data = [
        'title' => str_repeat('a', 255),
        'ingredients' => str_repeat('a', 255),
        'instructions' => str_repeat('a', 255),
    ];

    $this->actingAs(User::factory()->create())->post('/kitchen/recipes', $data);

    $this->assertDatabaseHas('recipes', $data);
}
```

## Conclusion

We've covered a breadth of Lucid features with this exercise, however it only covered the `Kitchen` service.
We are left with the Forum service, which is similar in process to Kitchen, just different business logic.
For that reason we'll leave it to you to test your familiarity with the architecture, and the full source code
is present as a reference <i class="fab fa-github fa-lg"></i>&nbsp;[Getting Started - Monolith](https://github.com/lucidarch/getting-started-monolith).

### Highlights

- Application logic is segregated in services that makes it extremely simple to find corresponding logic when needed.

- We are still using Laravel's internals and `artisan` for most of the things we did. This is intended to show how Lucid preserves
Laravel's defaults and avoids replicating or replacing them; in fact it complements them by elevating
their presence and fitting them within a defined structure.

- Navigating services and their features couldn't get any easier, by looking at the `app/Services/*/Features` directory you'd be able to have a summary of what
this application does at a glance.

- Visiting a feature's class would also provide an overview of the steps required with the least
details possible, yet details are available when wanting to dig deeper.
In addition to the presence of tests that mirror these classes which makes it easy to update functionality with reduced unintentional negative impact.

- Expanding on functionality such as `SaveRecipeJob` makes it easy to achieve a high degree of reusable code.
Consider updating a recipe, we'd still use this job to save an existing recipe by supplying an optional `id` parameter and have the
job create or update accordingly.
