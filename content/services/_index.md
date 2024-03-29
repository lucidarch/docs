---
title: "Services"
date: 2020-11-06T05:13:11Z
draft: false
weight: 15
hide: ["header"]
---

Are the differentiating factors of Monolith. Their presence is necessary in multi-purpose projects
that are set to scale in several areas of functionalities, *a.k.a horizonal growth*.
They communicate with domains to compose jobs into groups of functionalities in their own features and operations.

![Lucid Services](/media/images/services/services.png)

## Structure

Services are structured for scale, by adding an encapsulation layer for features, operations, routes, controllers, commands, resources (views)
providers, tests and database; allowing our application to contain sections that differ in high degree and have plenty of those without causing
clutter yet keep code navigation straight-forward.

The example below illustrates a comparison between a Lucid Monolith and a traditional directory separation in an application that
has Chat, Product Management, Forum, Api and Admin listed with their features below:

{{<columns>}}

- Chat
    - Send message
    - Update message
    - Delete message
    - Share message
    - Connect to channel
    - Search messages
- Forum
    - CRUD questions
    - Manage questions in categories
    - Schedule newsletter update
    - Set answer as accepted
    - Promote user as member
    - Follow/unfollow question
    - Search questions
    - SEO

<--->

- Product Management
    - CRUD product
    - Add product to category
    - Update product price
    - Update product availability
    - Search products
- Admin
    - CRUD users
    - CRUD organizations
    - CRUD products
    - CRUD projects
- Api
    - CRUD application (like Slack & Facebook apps etc.)
    - Send message
    - Subscribe to product updates
    - Subscribe to question updates

{{</columns>}}

An application this large will easily grow in files and directories to unmaintainable extents. To avoid that, we'll create services
for each to hold all their needs to function, which is exactly what's initially in Laravel's `/app` directory, in addition to tests.

{{%excerpt%}}
<pre>
app/Services/Chat
├── Console
│   └── Commands
├── <strong>Features</strong>
├── <strong>Operations</strong>
├── Http
│   ├── Controllers
│   └── Middleware
├── Providers
│   ├── ChatServiceProvider.php
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
{{%/excerpt%}}

{{% notice info %}}
{{<icon name="fa-info-circle">}}&nbsp;This is the initial structure of a service, however you may choose to customise it for what it needs only, for example if you
prefer to have the database stuff at the root or that this service doesn't have any database work to do on its own,
you would simply remove it from within the service. Similar for Console and everything else.
{{% /notice %}}

---

Here's an illustrative comparison between the traditional and the services approaches in routing and controllers:

### Routes Showcase

**Traditional**

Using the traditional approach where all of our routes are in `routes/web.php`, we'll find that this file will keep growing
the more functionalities we add. Also, if more than a single person is working on it, it is highly likely to produce conflicts
on merge which is always an unpleasant experience.

{{% notice light %}}
{{<icon name="fa-info-circle">}}&nbsp;Expand file locations to see their code.
{{% /notice %}}

<span class="lucid-code-expand">
{{% expand "routes/web.php" %}}
{{<gist mulkave 89eb6b9c6b0f9ab8574ab8e7a32aa8d2>}}
{{% /expand %}}
</span>

<span class="lucid-code-expand">
{{% expand "routes/api.php" %}}
{{<gist mulkave 316bf292318f16862103b248c363d38c>}}
{{% /expand %}}
</span>


Of course there are ways to separate routes into several files and load them in the service provider, but routing isn't
the only problem that we will be facing, it is just a part of it.

**Lucid Service Routes Showcase**

On the other hand, with services we've separated these routes files where they belong to have a pleasant experience working with them
and avoid conflicts as a team.

<span class="lucid-code-expand">
{{% expand "app/Services/Chat/routes/web.php" %}}

```php
Route::middleware(['auth', 'subscription'])->group('chat', function() {
    Route::post('/messages', [ChatController::class, 'sendMessage']);
    Route::put('/messages/{id}', [ChatController::class, 'updateMessage']);
    Route::post('/messages/{id}/share', [ChatController::class, 'shareMessage']);
    Route::post('/channels/{channel}/connect', [ChatController::class, 'connect']);
    Route::post('/search', [ChatController::class, 'search']);
});
```
{{% /expand %}}
</span>

<span class="lucid-code-expand">
{{% expand "app/Services/Forum/routes/web.php" %}}

```php
Route::middleware(['auth', 'subscription'])->group('forum', function() {
    Route::get('/questions', [ForumController::class, 'questions']);
    Route::post('/questions', [ForumController::class, 'addQuestion']);
    Route::put('/questions/{id}', [ForumController::class, 'updateQuestion']);
    Route::delete('/questions/{id}', [ForumController::class, 'deleteQuestion']);
    Route::put('/questions/{id}/category', [ForumController::class, 'category']);
    Route::put('/questions/{id}/accept', [ForumController::class, 'accept']);
    Route::post('/questions/{id}/follow', [ForumUserController::class, 'follow']);
    Route::get('/questions/search', [ForumUserController::class, 'search']);
    Route::put('/users/{id}/promote', [ForumUserController::class, 'promote']);
});
```
{{% /expand %}}
</span>

<span class="lucid-code-expand">
{{% expand "app/Services/Products/routes/web.php" %}}

```php
Route::middleware(['auth', 'subscription'])->group('products', function() {
    Route::get('/search', [ProductController::class, 'search']);
    Route::get('/', [ProductController::class, 'get']);
    Route::get('/{id}', [ProductController::class, 'show']);
    Route::post('/', [ProductController::class, 'add']);
    Route::put('/{id}', [ProductController::class, 'update']);
    Route::delete('/{id}', [ProductController::class, 'delete']);
    Route::put('/{id}/category', [ProductController::class, 'category']);
    Route::put('/{id}/price', [ProductController::class, 'price']);
    Route::put('/{id}/stock', [ProductController::class, 'stock']);
});
```
{{% /expand %}}
</span>

<span class="lucid-code-expand">
{{% expand "app/Services/Api/routes/api.php" %}}

```php
Route::middleware(['throttle:api'])->group('api', function () {
    Route::get('/apps/search', [AppController::class, 'search']);
    Route::get('/apps', [AppController::class, 'get']);
    Route::get('/apps/{id}', [AppController::class, 'show']);
    Route::post('/apps', [AppController::class, 'add']);
    Route::put('/apps/{id}', [AppController::class, 'update']);
    Route::delete('/apps/{id}', [AppController::class, 'delete']);
    Route::post('/channels/{id}', [ChatController::class, 'subscribe']);
    Route::post('/products/{id}', [ProductController::class, 'subscribe']);
    Route::post('/questions/{id}', [ForumController::class, 'subscribe']);
});
```
{{% /expand %}}
</span>

<span class="lucid-code-expand">
{{% expand "app/Services/Admin/routes/web.php" %}}
{{<gist mulkave 37bb21541cf42f2cafa6f3fe98c8e0c3>}}
{{% /expand %}}
</span>

Another variation of routes would be to reduce Admin routes by including them in their corresponding services instead of having
Admin service include them all. Both approaches have their benefits so you may choose whichever works best for your case.

<span class="lucid-code-expand">
{{% expand "app/Services/Products/routes/web.php" %}}
{{<gist mulkave 72777eabde48996067daf968b62c468f>}}
{{% /expand %}}
</span>


---

### Controllers Showcase

Consolidating routes and controllers in services allows us to have them side-by-side within the same context, so that we don't
have to go digging for code in foreign directories.

{{<columns>}}

**Traditional**

<pre>
app/Http/Controllers
├── Admin
│   ├── ProductController.php
│   ├── ForumController.php
│   ├── UserController.php
│   └── OrganizationController.php
├── Api
│   ├── AppController.php
│   ├── ChatController.php
│   └── HookController.php
├── Chat
│   └── MessageController.php
├── Products
│   └── ProductController.php
└── Forum
    ├── QuestionController.php
    ├── CategoryController.php
    └── MemberController.php
</pre>

<--->

**Lucid Service Controllers**

<pre>
app/Services/Admin/Http/Controllers
   ├── ProductController.php
   ├── ForumController.php
   ├── UserController.php
   └── OrganizationController.php

app/Services/Api/Http/Controllers
   ├── AppController.php
   ├── ChatController.php
   └── HookController.php

app/Services/Chat/Http/Controllers
   └── MessageController.php

app/Services/ProductManagement/Http/Controllers
   └── ProductController.php

app/Services/Forum/Http/Controllers
    ├── QuestionController.php
    ├── MemberController.php
    └── CategoryController.php
</pre>

{{</columns>}}

---

## Benefits

{{<columns>}}

{{% panel %}}
### Concentration

Working on a feature in a service wouldn't concern you with others outside the bounds of said service.
{{% /panel %}}

<--->

{{% panel %}}
### Efficiency

Reduced code review time and merge conflicts when working with a team, due to the separation of concerns.
{{% /panel %}}

<--->

{{% panel %}}
### Separation

Consider having Single Responsibility and Separation of Concerns, not only at the code level, but also in the structure.
{{% /panel %}}

{{</columns>}}

{{%notice info%}}
{{<icon name="fa-asterisk">}}&nbsp;Remember that it is completely optional to use Services. In fact there is no obligation
whatsoever with Lucid, pick any unit from the stack and use it to your convenience; just preserve Lucid's guidelines as you do.
{{%/notice%}}

## Create a Service

```bash
lucid make:service Chat
```

This will generate the directory structure for a service in `app/Services/Chat/*`

{{% excerpt-include filename="/services/_index.md" /%}}

### Register Service

Once created we'll need to tell Laravel about our service so that it loads its files such as routes, migrations, views and others.
There are two ways to register the service's provider, in this case it's `ChatServiceProvider`:

**1. In Configuration: `config/app.php`**

Add `App\Services\Chat\Providers\ChatServiceProvider::class` to `'providers'` in `config/app.php`

This will register and load the service every time the application launches.

**2. In `AppServiceProvider::register`**

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use App\Services\Api\Providers\ApiServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->register(ApiServiceProvider::class);
    }
}
```

This opens up possibilities for conditional registration of services. For example you may choose to load Dashboard service
only when in the local environment:

```php
public function register()
{
    if (App::environment('local')) {
        $this->app->register(DashboardServiceProvider::class);
    }
}
```

### Run Service Tests

Initially service tests are part of the default testsuite, but in case you wish to run a single service's tests in isolation
please add the following to your `phpunit.xml` under `<testsuites>` (must be done for each service),
and then we would be able to use `phpunit --testsuite <name>` to run them.

```xml
<testsuite name="Chat">
    <directory suffix="Test.php">./tests/Feature/Services/Chat</directory>
    <directory suffix="Test.php">./tests/Unit/Services/Chat</directory>
</testsuite>
```

Now we can run the tests for this service in isolation:

```bash
phpunit --testsuite Chat
```

## Working with Services

All unit generation commands (`lucid make:*`) have an optional parameter `<service>` to specify which service you'd like to generate the unit into.

{{% notice light %}}
{{<icon name="fa-info-circle">}}&nbsp;command parameters are case-insensitive, Lucid will try to convert to the corresponding case.
{{% /notice %}}

### Controllers

**Signature** `lucid make:controller <controller> <service>`

**Example**

```bash
lucid make:controller message chat
```

Will generate `app/Services/Chat/Http/Controllers/MessageController.php`

```php
<?php

namespace App\Services\Chat\Http\Controllers;

use Lucid\Units\Controller;

class MessageController extends Controller
{

}
```

### Features

**Signature** `lucid make:feature <feature> <service>`

**Example**

```bash
lucid make:feature SendMessage chat
```

Will generate `app/Services/Chat/Features/SendMessageFeature.php`

```php
<?php

namespace App\Services\Chat\Features;

use Lucid\Units\Feature;
use Illuminate\Http\Request;

class SendMessageFeature extends Feature
{
    public function handle(Request $request)
    {

    }
}

```

### Operations

**Signature** `lucid make:operation <operation> <service>`

**Example**

```bash
lucid make:operation EnqueueMessageForSending chat
```

Will generate `app/Services/Chat/Operations/EnqueueMessageForSendingOperation.php`

```php
<?php

namespace App\Services\Chat\Operations;

use Lucid\Units\Operation;

class EnqueueMessageForSendingOperation extends Operation
{
    /**
     * Create a new operation instance.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Execute the operation.
     *
     * @return void
     */
    public function handle()
    {

    }
}

```

### Migrations

When generated, migrations are automatically registered in the global application's migrations so Laravel knows about them
and running them can be done simply by using `php arisan migrate`. This is done in the Service's
service provider class (in our example it's `ChatServiceProvider`) that we added to our app's `providers` upon creation.

#### Generate Migration

**Signature** `lucid make:migration <migration> <service>`

**Example**

```bash
lucid make:migration create_messages_table chat
```

Will generate `app/Services/Chat/database/migrations/{date}_create_messages_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateMessagesTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('messages', function (Blueprint $table) {
            $table->id();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('messages');
    }
}
```

#### Run Service Migrations

Our migrations are recognized by Laravel due to them being loaded automatically in the service provider - `ChatServiceProvider` in this example.

Which makes it straight-forward to run them:

```bash
php artisan migrate
```

#### Removing Migrations

In the case where your service doesn't use any of the database functionalities and you'd like to tidy up by deleting the `database`
folder, make sure you remove the following from the service provider - typically at `app/Services/{name}/Providers/{name}ServiceProvider.php`

```php
public function boot()
{
    $this->loadMigrationsFrom([
        realpath(__DIR__ . '/../database/migrations')
    ]);
}
```

### Lang & Views

Each service is provided with its own `resources` directory that resembles Laravel's, containing `views` and `lang`.
All the files that are created there will be registered under a namespace named in snake_case after the service's name.

This is initially done in the Service Provider upon creating the service, for example in `ChatServiceProvider::registerResources`:

```php
protected function registerResources()
{
    // Translation must be registered ahead of adding lang namespaces
    $this->app->register(TranslationServiceProvider::class);

    Lang::addNamespace('chat', realpath(__DIR__.'/../resources/lang'));

    View::addNamespace('chat', base_path('resources/views/vendor/chat'));
    View::addNamespace('chat', realpath(__DIR__.'/../resources/views'));
}
```

To use these views simply prepend them with their corresponding namespace `{namespace}::{view}`.

```php
Route::get('/chat', function() {
    return view('chat::welcome');
});
```

For multi-word services such as `ProductManagement` it will be `product_management::welcome`.

If your service doesn't use views and you'd like to de-clutter, feel free to remove the `resources` folder altogether and
remember to remove `registerResources` method from the service provider as well as its call in `register`.

### Tests

Testing services is about testing their features and operations, they're scoped according to the type of test:
- Feature: `tests/Feature/Services/{service}/{Feature}Test.php`
- Operation: `tests/Unit/Services/{service}/{Operation}Test.php`

Other tests that belong to the service can be placed in correspondance to the above.

It is also recommended to namespace your test classes just to avoid conflicts. This is done by default for features and operations
that are generated using `lucid make:*` following PSR-4 standard. An initial test could look like this:

```php
<?php

namespace Tests\Feature\Services\Chat;

use Tests\TestCase;
use App\Services\Chat\Features\SendMessageFeature;

class SendMessageFeatureTest extends TestCase
{
    public function test_sendmessagefeature()
    {
        $this->markTestIncomplete();
    }
}
```

### Providers

Service providers are the connecting wires between the Lucid stack and Laravel especially when it comes to services,
they are used to tell Laravel where our code is and what to load, from where, and when.

Each service comes with its own set of providers in `app/Services/{service}/Providers` and most essentially the service's own
provider that is usually named after the service *e.g. `ChatServiceProvider` or `ProductManagementServiceProvider`*.

For Laravel to know about our service and load its files such as database migrations, lang, views and any other classes and elements
you would like to register in that service specifically, the service provider must be part of `providers` in `config/app.php`

```php
'providers' => [
    ...
    App\Services\Api\Providers\ApiServiceProvider::class,
    App\Services\Chat\Providers\ChatServiceProvider::class,
    App\Services\ProductManagement\Providers\ProductManagementServiceProvider::class,
]
```

#### Custom Service Provider

We can create and register our own providers which allows us to segregate functionality further within our services:

`app/Services/Chat/Providers/RiakServiceProvider`
```php
<?php

namespace App\Services\Chat\Providers;

use Riak\Connection;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection(config('riak'));
        });
    }
}
```

Then register `RiakServiceProvider` in the service's provider to be automatically loaded whenever the service's provider is loadeo:

`app/Services/Chat/Providers/ChatServiceProvider`

```php
public function regsiter()
{
    $this->app->register(RiakServiceProvider::class);
}
```

## A Note On Microservices

Talking about projects at scale may often lead us to talk about Microservices. Turning your Monolith services into
several instances of their own is one of the ways you may benefit from using services in a Monolith since each resembles a
Laravel project, and naturally in microservies we'd create a separate Laravel
installation for each microservice, our Lucid services are ready for that due to the correspondence in structure;
nevertheless it would still require considerable effort to make the move but it is reduced significantly with this in hand,
especially that there will be much more to consider in the transformation.

{{% notice info %}}
This is likely to occur in a number of projects but certainly not all of them.
{{% /notice %}}

In Lucid terms: each of the services will become a Micro instance, preserving features & operations with their entrypoints and their tests.
What will be left as a dependency is domains and data which can be created in a package and shared among the microservices that require it.
Even though you may wish to also have proprietary domains in the services themselves which will be possible to do as well.

Visit ["Micro vs. Monolith"]({{%ref "/micro-vs-monolith"%}}) for more.

## SOA & Lucid

When the word "services" is mentinoed it is invevitable to drift in thought towards Service-Orienter Architecture,
where controllers call a method from a "service" class that performs the work required. In most cases where this structure
is used, it was found that eventually services became the new controllers - large and full of unmaintainable logic.
In addition to the difficulty in sharing code between services due to the tight coupling with other components.

For that reason, Lucid services take it up a level to give the service a structure that allows it to grow, by organize its code
according to a familiar structure and expose its functionalities.

### Are Lucid Services Complient to SOA?

In brief, yes. However, there's a difference in the hierarchy:

- In SOA, the application layer is at the top, it includes the services that are needed and the application registers
and handles the routing, controller and directs the requests to the service's entrypoint.
- Lucid Services on the other hand give that power to the service so that it implements all that is required from routes, controllers,
and the rest, and the application will only register the service.
This approach has proven to be efficient at scale due to the degree of separation, specifically with team collaboration, interoperability
and integrity.

{{<figure alt="Lucid Service & Service-Oriented Architecture" src="/media/images/services/soa.png" width="500">}}
