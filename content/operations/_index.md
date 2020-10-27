---

title: "Operations"
date: 2020-10-20T10:42:27Z
draft: true
weight: 8
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
        $author = $this->run(GetAuthorByIDJob::class, [
            'id' => $this->authorId,
        ]);

        do {

            $result = $this->run(PaginateSubscribersJob::class, [
                'authorId' => $this->authorId,
            ]);

            if ($result->subscribers->isNotEmpty()) {
                // it's a queueable job so it will be enqueued, no waiting time
                $this->run(SendNotificationJob::class, [
                    'from' => $author,
                    'to' => $result->subscribers,
                    'notification' => 'article.published',
                ]);
            }

        } while ($result->next !== null);

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
$this->run(NotifySubscribersOperation::class, [
    'authorId' => $authorId,
]);
```

## Generate Operation Class

Use `lucid` CLI to generate an Operation class that extends Lucid's Operation base class by default, which `handle` method is invoked when run by a Feature or a custom dispatcher.

{{% tabs %}}

    {{% tab "Micro" %}}
**Signature**

```bash
lucid make:operation <operation> {--Q|queue}
```

**Example**
```bash
lucid make:operation NotifySubscribers
```
Generated class will be at `app/Operations/NotifySubscribersOperation.php`

and its test at `tests/Operations/NotifySubscribersOperationTest.php`
    {{% /tab %}}

    {{% tab "Monolith" %}}
**Signature**

```bash
lucid make:operation <operation> <service> {--Q|queue}
```

**Example**

```bash
lucid make:operation NotifySubscribers publishing
```

Generated class will be at `src/Services/Publishing/Operations/NotifySubscribersOperation.php`

and its test at `src/Services/Publishing/Tests/Operations/NotifySubscribersOperationTest.php`

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

   $this->run(SetArticlePublishingRulesOperation::class, [
        'id' => $request->input('id'),
        'schedule' => $request->input('datetime'),
        'platforms' => $request->input('platforms'),
        'visibility' => $request->input('visibility'),
   ]);

   $this->run(NotifySubscribersOperation::class, [
        'authorId' => Auth::id(),
   ]);

   $article = $this->run(new GetArticleByIDJob($request->input('id')));

   return $this->run(RespondWithViewJob::class, [
        'data' => compact('article'),
        'template' => 'articles.publish.success',
   ]);
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
- `tests/Operations/NotifySubscribersOperationTest.php`

    {{% /tab %}}

    {{% tab Monolith %}}

```bash
lucid make:operation NotifySubscribers publishing
```

Would generate two files:

- `src/Services/Publishing/Operations/NotifySubscribersOperation.php`
- `src/Services/Publishing/Tests/Operations/NotifySubscribersOperationTest.php`

    {{% /tab %}}

{{% /tabs %}}

Let's test our operation:

```php

```
