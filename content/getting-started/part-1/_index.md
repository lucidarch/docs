---
title: "Part 1"
date: 2020-10-13T14:54:00Z
draft: true
weight: 2
hide:
- toc
---

## 1. Create Api Service

We will create a service called `Api`. This service, obviously allows Api access to our application. Another example of a service would be `Admin` where the application can be managed through an administration interface, or `Web` for the website of this application.

```bash
lucid make:service Api
```

### Register Api

- Open `src/Foundation/Providers/ServiceProvider`
- Add `use App\Services\Api\Providers\ApiServiceProvider` to the top
- In the `register` method add `$this->app->register(ApiServiceProvider::class)`

Now our project recognises Api service, so we can add Features and Jobs to get things moving.

The Api directory will initially contain the following:

```bash
src/Services/Api
├── Console         # Everything that has to do with the Console (i.e. Commands)
├── Features        # Contains the Api's Features classes
├── Http            # Routes, controllers and middlewares
├── Providers       # Service providers and binding
├── database        # Database migrations and seeders
└── resources       # Assets, Lang and Views
```

## 2. CreateArticleFeature

Using the CLI's `make:feature {feature} {service}` we will create our first Feature

```bash
lucid make:feature CreateArticle api
```

`CreateArticle` will eventually be created as `CreateArticleFetaure`. Even if you had entered `CreateArticleFeature` or `createArticleFeaure` it would still work. However, middle words are case-sensitive, so `createarticle` would have ended up `CreatearticleFeature`.

Two new files have been created with this command:

`src/Services/Api/Features/CreateArticleFeature.php` and `src/Services/Api/Tests/CreateArticleFeatureTest.php`

- Open `src/Services/Api/Features/CreateArticleFeature.php` file to fill the `handle` method with the steps that we are about to run. This method is executed automatically when calling `serve(CreateArticleFeature::class)` inside the controller as we will see later.

```php
namespace App\Services\Api\Features;

use Lucid\Foundation\Feature;
use Illuminate\Http\Request;

class CreateArticleFeature extends Feature
{
    public function handle(Request $request)
    {
        // Validate article input

        // Save article

        // Respond with JSON
    }
}
```

Each of these comments inside the `handle` method means that we need to create a job to perform its task.

### 2.1 ValidateArticleInputJob

Our first job is to validate the input. As promised, Lucid will provide the clarity of reading your task list when overlooking the Feature's `handle` class. For that reason, our job's class will be called exactly what needs to be done `ValidateArticleInputJob`, to be create using the command `make:job` with the following signature:

```bash
lucid make:job {job} {domain}
```

Our job will live inside the `Article` domain, which will contain everything related to managing articles:

```bash
lucid make:job ValidateArticleInput article
```

This will generate two files:

`src/Domains/Article/Jobs/ValidateArticleInputJob.php` and `tests/Domains/Article/Jobs/ValidateArticleInputJobTest.php`

Let's examine our job class:

```php
class ValidateArticleInputJob extends Job
{
    /**
     * Create a new job instance.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Execute the job.
     *
     * @return void
     */
    public function handle()
    {
        //
    }
}
```

The `__construct` defines the signature of the job, it defines what the job requires to be run. `handle` is the method that is automatically called when the feature calls `$this->run(ValidateArticleInputJob::class)`

#### Fill Validation Job

Let's retrieve the input and validate it here. The construct will receive the input as received from the request and will pass it to the validator which will throw an exception if it doesn't qualify, and return `true` if it does.

For the simplicity of this example we chose to add the generic validator and perform the validation in the job itself, however, a better approach would be to create a validator [or more] per domain.

PHPDoc Blocks were eliminated for brevity, but they are recommended to be kept and updated, as they do exist in the code shared on GitHub.

```php
namespace App\Domains\Article\Jobs;

use Lucid\Foundation\Job;
use Lucid\Foundation\Validator;

class ValidateArticleInputJob extends Job
{
    private $input;

    private $rules = [
        'title' => ['required', 'string', 'max:100'],
        'content' => ['required', 'string', 'max:1000'],
    ];

    public function __construct(array $input)
    {
        $this->input = $input;
    }

    public function handle(Validator $validator) : bool
    {
        return $validator->validate($this->input, $this->rules);
    }
}
```

The `handle` method supports `IoC` so we can inject classes and have them resolved. We are also using Lucid's `Validator` class for easy validation.

#### Test Validation Job

We love tests, for that reason it is recommended at this stage to write a test to ensure that this job does exactly what is intended in `tests/Domains/Article/Jobs/ValidateArticleInputJobTest.php`

**Test Validation Success Scenario**

As an initial test, let's ensure that all goes well within our job:

```php
public function test_validate_article_input_job()
{
    $job = new ValidateArticleInputJob([
        'title' => 'The Title',
        'content' => 'The content of the article goes here.',
    ]);

    $this->assertTrue($job->handle(app(Validator::class)));
}
```

Running this test should go all green. Now let's test our validation failure cases using a [data provider](https://phpunit.de/manual/3.7/en/appendixes.annotations.html#appendixes.annotations.dataProvider)

**Test Validation Failure Scenarios**

Using Lucid's `Validator` the exception that will be thrown is `\Lucid\Foundation\InvalidInputException` which is surely customisable by simply extending the `Validator` class with your own and providing a custom exception to be thrown which is explained in depth in [..... LINK TO VALIDATOR EXPLANATION ....].

The test will look as follows:

```php
/**
 * @dataProvider articleInputValidationProvider
 * @expectedException \Lucid\Foundation\InvalidInputException
 */
public function test_validating_article_input_job_rules($title = null, $content = null)
{
    $job = new ValidateArticleInputJob(compact('title', 'content'));
    $job->handle(app(Validator::class));
}

public function articleInputValidationProvider()
{
    return [
        'without title' => [
            'content' => 'The content of the article.',
        ],
        'title is empty' => [
            'title' => '',
            'content' => 'The content of the article.',
        ],
        'without content' => [
            'title' => 'The Title Only',
        ],
        'content is empty' => [
            'title' => 'The Title Here',
            'content' => '',
        ],
        'max title length' => [
            'title' => str_repeat('a', 101),
            'content' => 'Content goes here',
        ],
        'max content length' => [
            'title' => 'Title here',
            'content' => str_repeat('a', 1001),
        ],
    ];
}
```

There sure are better ways to deal with data providers, but for the simplicity of this example it was decided to use them as such.

### 2.2 SaveArticleJob

Our next task is to store the article information in the database. Simple.

Generate `SaveArticleJob` in our article domain

```bash
lucid make:job SaveArticle article
```

Before we start filling our job, we need to ensure that the `Article` object exists. For the sake of simplicity, we will not be using the `Article` object that is shipped with Laravel, but we will create our own with the `lucid make:model` command:

```bash
lucid make:model Article
```

A new `Article` class has been created under `src/Data/Article.php`, let's fill it:

```php
namespace App\Data;

use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    protected $fillable = ['title', 'content'];
}
```

#### Prerequisites

Before continuing with this example we need to configure a database to save our articlr into. For this example we will use `sqlite` for we will only be running our code through tests.

- Configure PHPUnit to use SQLite by adding the following to the bottom of `phpunit.xml` file under the `<php>` tag:

    ```xml
    <server name="DB_CONNECTION" value="sqlite"/>
    <server name="DB_DATABASE" value=":memory:"/>
    ```

- Generate migration to create articles table

    ```bash
    lucid make:migration create_articles_table api
    ```

    Like other Lucid components, migrations are also service-specific. More on this in the Data section [[.......... LINK TO LUCID DATABASE SECTION ..........]]. This will generate a new file in `src/Services/api/database/migrations`

    ```php
    use Illuminate\Support\Facades\Schema;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;

    class CreateArticlesTable extends Migration

        public function up()
        {
            Schema::create('articles', function (Blueprint $table) {
                $table->bigIncrements('id');
                $table->string('title', 100);
                $table->string('content', 1000);
                $table->timestamps();
            });
        }

        public function down()
        {
            Schema::dropIfExists('articles');
        }
    }
    ```

#### Save the Article

In our job `src/Domains/Article/Jobs/SaveArticleJob.php` we read the values required for this task and simply create an `Article` [Eloquent](https://laravel.com/docs/6.x/eloquent) instance and save it.

```php
namespace App\Domains\Article\Jobs;

use App\Data\Article;
use Lucid\Foundation\Job;

class SaveArticleJob extends Job
{
    private $title;

    private $content;

    public function __construct(string $title, string $content)
    {
        $this->title = $title;
        $this->content = $content;
    }

    public function handle() : Article
    {
        $article = new Article([
            'title' => $this->title,
            'content' => $this->content,
        ]);

        $article->save();

        return $article;
    }
```

You might be thinking why we didn't create a "CreateArticleJob" instead of this. The answer to this is reusability, for later on we will need to probably update an article or expand on the functionality of saving an article into the database, so this job's functionality will be reused.

Another question that comes to mind here is: why do we receive `title` and `content` and not just pass the input as received from the request as an array, or even the `Request` itself, which can even be injected in `handle`?

And its tests to ensure it works:

Note the included traits `DatabaseMigrations` and `RefreshDatabase`.

```php
namespace App\Domains\Article\Tests\Jobs;

use Tests\TestCase;
use App\Data\Article;
use App\Domains\Article\Jobs\SaveArticleJob;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\DatabaseMigrations;

class SaveArticleJobTest extends TestCase
{
    use RefreshDatabase;
    use DatabaseMigrations;

    public function test_create_article_job()
    {
        $job = new SaveArticleJob('My Article', 'Lorem ipsum dolor sit amet');

        $article = $job->handle();

        $this->assertInstanceOf(Article::class, $article);
        $this->assertIsInt($article->id);
        $this->assertEquals('My Article', $article->title);
        $this->assertEquals('Lorem ipsum dolor sit amet', $article->content);
    }
}
```

### 2.3 Serving Features

Back to our Feature class `CreateArticleFeature` , now that we are sure that our jobs will run as expected and return the expected results, and given that we provide the correct input, we now need to run them in the `handle` method of our Feature:

**Note:** `RespondWithJsonJob` is a built-in job so no need to do anything additional to have it.

```php
namespace App\Services\Api\Features;

use Illuminate\Http\Request;
use Lucid\Foundation\Feature;
use App\Domains\Article\Jobs\SaveArticleJob;
use App\Domains\Http\Jobs\RespondWithJsonJob;
use App\Domains\Article\Jobs\ValidateArticleInputJob;

class CreateArticleFeature extends Feature
{
    public function handle(Request $request)
    {
        $this->run(ValidateArticleInputJob::class, [
            'input' => $request->input(),
        ]);

        $article = $this->run(SaveArticleJob::class, [
            'title' => $request->input('title'),
            'content' => $request->input('content'),
        ]);

        return $this->run(new RespondWithJsonJob($article));
    }
}
```

Our jobs have replaced the comments that was previously present here because it would be redundant to be kept; this highlights the importance in naming your jobs properly and being as expressive as you would in the comment that would've replaced it to explain its role.

Let's examine the Feature above before we head to its tests.

**A Glance at the Feature**

At a glance, you can tell the following:

- The steps that are required to accomplish this feature
- What each of the steps require as input in order to run
- Not much clutter, just enough information to be introduced to the feature

**Feature →Run→ job Syntax**

Mark the syntax of running the validation job:

```php
$article = $this->run(SaveArticleJob::class, [
    'title' => $request->input('title'),
    'content' => $request->input('content'),
]);
```

This syntax enables us to pass parameters interchangeably, without caring about the order in which they are defined within the Job's signature. For example, the following would still work:

```php
$article = $this->run(SaveArticleJob::class, [
    'content' => $request->input('content'),
    'title' => $request->input('title'),
]);
```

This grants us the isolation required for each scope:

- At the Job level, I am free to change the signature as I see fit
- At the Feature level, I am only concerned with the required parameters, not their order

**Testing the Feature**

To ensure the validity of the above, we sure have to write a test to execute our Feature.

Testing Features is different from that of Jobs and Operations. It is a functional test that ensures the integration and integrity of all the Jobs and Operations that are run by the feature and it is done using [Laravel's HTTP tests](https://laravel.com/docs/6.x/http-tests). The test file for our Feature has already been created so we just need to fill it:

```php
namespace App\Tests\Features;

use Tests\TestCase;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\DatabaseMigrations;

class CreateArticleFeatureTest extends TestCase
{
    use RefreshDatabase;
    use DatabaseMigrations;

    public function test_create_article_feature()
    {
        $title = 'Getting Started With The Lucid Architecture';
        $content = 'Take a deep breath, make a smooth coffee and get rolling!';

        $response = $this->post('/api/articles', compact('title', 'content'));

        $response->assertOk();
        $response->assertJson([
            'status' => 200,
            'data' => [
                'title' => $title,
                'content' => $content,
            ]
        ]);
    }
}
```

This will ensure that the Feature works, but it it's sufficient for us to sleep in peace at night, because it doesn't cover error cases to ensure that the client of our Api service will receive an informative error response in JSON format. We will expand on this in Part III of this tutorial.
