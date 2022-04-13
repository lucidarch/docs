---

title: "Operations"
date: 2020-10-20T10:42:27Z
draft: false
weight: 14
hide: ["header"]

---

Their purpose is to increase the degree of code reusability by piecing jobs together to provide composite functionalities from across domains.

---

Operations are a group of jobs that deliver multi-step functionalities.
Technically, Operation classes are similar to Feature classes in usage; meaning that they both have
`run($job,$params)` in common to run jobs from any domain, and can be called standalone (e.g. call an operation from a Command).

However, conceptually they have their differences in that a Feature can run multiple Operation and Job classes while an Operation can run multiple Jobs only.
They also differ in what they represent to the application: A feature is what the application provides to the outside, while an operation is more of an internal aspect. Technically they differ in they way they are dispatched, where we `serve` a feature but we `run` an operation;
any class can do so the same as a Feature class would by turning it into a [dispatcher class]({{% ref "/jobs/#custom-dispatcher" %}} "Custom Dispatcher").

**Example**

Given that we are working on a publishing platform, and upon creating an article we would like to send notifications to the subscribers
of the author, here's what that operation may look like:

```php
class NotifySubscribersOperation extends Operation
{
    private int $authorId;

    public function __construct(int $authorId)
    {
        $this->authorId = $authorId;
    }

    /**
     * Sends notifications to subscribers.
     *
     * @return int Number of notification jobs enqueued.
     */
    public function handle(): int
    {
        $author = $this->run(new GetAuthorByIDJob(
            id: $this->authorId,
        ));

        do {

            $result = $this->run(new PaginateSubscribersJob(
                authorId: $this->authorId,
            ));

            if ($result->subscribers->isNotEmpty()) {
                // it's a queueable job so it will be enqueued, no waiting time
                $this->run(new SendNotificationJob(
                    from: $author,
                    to: $result->subscribers,
                    notification: 'article.published',
                ));
            }

        } while ($result->hasMorePages());

        return $result->total;
    }
}
```

As you see the jobs that are used in this operation are ones that can be shared with other areas in our code as well,
increasing the degree of reusable code.

1. `GetAuthorByIDJob`: to retrieve a user/author by ID is as abstract as it can get, and will definitely be used numerous times in our application.
2. `PaginateSubscribersJob`: would be used every time we need to retrieve an author's subscribers. Such jobs usually grow in responsibility over time
and their results become more customizable.

    *Example: later it may allow to specify a certain limit to the number of subscribers to return,
then we'd be able to paginate them for listing subscribers in a view or over an API.*

3. `SendNotificationJob`: will be used every time we need to send a notification, regardless of the type of notification to be sent
since it is specified with the `notification` parameter. Which also can grow into providing more customization such as specifying
the type of notification to send (e.g. mobile, browser, web, email etc.).

With this we've achieved **single responsibility** at a **low cost of debt** and **prepared for scale**. In fact,
some of these jobs would've been implemented already by the time we reached this operation which makes it **quick to biuld**.

Now we have that functionality bundled at our disposal to be called whenever needed:

```php
$this->run(new NotifySubscribersOperation(
    authorId: $authorId,
));
```

## Generate Operation Class

Use `lucid` CLI to generate an Operation class that extends Lucid's Operation base class by default, which `handle` method is invoked when run by a Feature or a custom dispatcher.

{{% tabs %}}

    {{% tab "Micro" %}}
**Signature** `lucid make:operation <operation> {--Q|queue}`

**Example**
```bash
lucid make:operation NotifySubscribers
```
Generated class will be at `app/Operations/NotifySubscribersOperation.php`

and its test at `tests/Unit/Operations/NotifySubscribersOperationTest.php`
    {{% /tab %}}

    {{% tab "Monolith" %}}
**Signature** `lucid make:operation <operation> <service> {--Q|queue}`

**Example**

```bash
lucid make:operation NotifySubscribers publishing
```

Generated class will be at `app/Services/Publishing/Operations/NotifySubscribersOperation.php`

and its test at `tests/Unit/Services/Publishing/Operations/NotifySubscribersOperationTest.php`

    {{% /tab %}}

{{% /tabs %}}

The generated Operation class will automatically be suffixed with `Operation`, so there's no need for it to be specified in the command.

## Calling Operations

Similar to jobs, and Operation can also be called using the `run` method that's provided by extending one Lucid's `Feature` or a [custom dispatcher](/jobs/#custom-dispatcher),

See [here]({{% ref "/jobs/#calling-jobs"%}}) for more on the `run` method.

```php
class PublishArticleFeature extends Feature
{
   $this->run(new ValidateArticlePublishingInputJob($request->input()));

   $this->run(new SetArticlePublishingRulesOperation(
        id: $request->input('id'),
        schedule: $request->input('datetime'),
        platforms: $request->input('platforms'),
        visibility: $request->input('visibility'),
   ));

   $this->run(new NotifySubscribersOperation(
        authorId: Auth::id(),
   ));

   $article = $this->run(new GetArticleByIDJob($request->input('id')));

   return $this->run(new RespondWithViewJob(
        data: compact('article'),
        template: 'articles.publish.success',
   ));
}
```

## Queueable Operations

You may turn any operation into a queueable operation that will be dispatched using [Laravel Queues](laravel.com/docs/queues)
rather than running synchronously, by simply implementing `ShouldQueue` interface.

### Generate Queueable Operation

Use the `--queue` or shorthand `-Q` to generate a queueable operation.

```bash
lucid make:job NotifySubscribers --queue
```

Will produce the following operation class:

```php
class NotifySubscribersOperation extends Operation implements ShouldQueue
{
    public function handle()
    {
        // notifications processing will happen in the queue
    }
}
```

This job will be treated exactly as Laravel treats [queued jobs](https://laravel.com/docs/queues#class-structure).

### Specify Queue Name

```php
public function __construct()
{
    /*
     * set the name of the queue on which to dispatch this operation.
     * if using Horizon, this should be the same as the one configured there.
     */
    $this->onQueue('notifications');
}
```

## Testing

When generating an operation with `lucid make:operation` a test class is automatically generated along, having `markIncomplete()` statement in a stub method as a reminder to fill the missing test.


{{% tabs %}}

    {{% tab Micro %}}

```bash
lucid make:operation NotifySubscribers
```

Would generate two files:

- `app/Operations/NotifySubscribersOperation.php`
- `tests/Unit/Operations/NotifySubscribersOperationTest.php`

    {{% /tab %}}

    {{% tab Monolith %}}

```bash
lucid make:operation NotifySubscribers publishing
```

Would generate two files:

- `app/Services/Publishing/Operations/NotifySubscribersOperation.php`
- `tests/Unit/Services/Publishing/Operations/NotifySubscribersOperationTest.php`

    {{% /tab %}}

{{% /tabs %}}

The purpose of operation testing is to ensure that the integration between the jobs it runs is working as expected,
but we do not have to test every job's case on its own, for that we rely on jobs being tested for their integrity.

For example, consider the following operation test:

```php
<?php

namespace Tests\Unit\Services\Publishing\Operations;

use Tests\TestCase;
use App\Data\Models\Author;
use App\Data\Models\Subscriber;
use Illuminate\Support\Facades\Queue;
use App\Services\Operations\NotifySubscribersOperation;

class NotifySubscribersOperationTest extends TestCase
{
    public function test_successfully_notifying_subscribers()
    {
        // SendNotificationJob will be dispatched to the queue
        Queue::fake();

        // queue must be empty
        Queue::assertNothingPushed();

        // n. of subscribers we're testing with
        $subscribers = 10;

        $author = Author::factory()
            ->has(Subscriber::factory($subscribers));
            ->create();

        $op = new NotifySubscribersOperation($author->id);
        $result = $op->handle();

        // assert all subscribers were paginated
        $this->assertEquals($subscribers, $result);

        // assert the correct n. of SendNotificationJob were dispatched
        Queue::assertPushed(SendNotificationJob::class, $subscribers);
    }
}
```

### Mocking Jobs

When testing, you may occasionally need to skip dispatching a certain job but would still want to make sure that the operation
actually ran the job as expected, with the correct parameters.
In such cases we would mock the operation's `run` [partially](http://docs.mockery.io/en/latest/reference/partial_mocks.html "Partial Mocks").

In our case we'd update our test to not dispatch `SendNotificationJob` so that we don't actually send notifications.
This may seem odd at first because we are mocking the class that we are actually testing,
but **with partial mocks only the methods that we set expectations on would be mocked and the rest would be dispatched.**
And in case the operation doesn't call `run(SendNotificationJob::class, $params)` with the expected parameters the test will fail.

```php
<?php

namespace Tests\Unit\Services\Publishing\Operations;

use Mockery;
use Tests\TestCase;
use App\Data\Models\Author;
use App\Data\Models\Subscriber;
use App\Services\Operations\NotifySubscribersOperation;

class NotifySubscribersOperationTest extends TestCase
{
    public function test_successfully_notifying_subscribers_with_mock()
    {
        // n. of subscribers we're testing with
        $subscribers = 10;

        $author = Author::factory()
            ->has(Subscriber::factory($subscribers));
            ->create();

        // create operation mock instance
        $mOp = Mockery::mock(NotifySubscribersOperation::class, [$author->id]);

        // set expectations to jobs that need to be skipped
        $mOp->shouldReceive('run')
            ->with(SendNotificationJob::class, [
                'from' => $author,
                'to' => $author->subscribers,
                'notification' => 'article.published',
            ]);

        $result = $mOp->handle();

        // assert all subscribers were paginated
        $this->assertEquals($subscribers, $result);
    }
}
```

Whether to mock or not is a case-by-case decision, but as a general guideline it is best to always test with what's closest to reality.
