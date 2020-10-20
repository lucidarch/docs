---

title: "Operation"
date: 2020-10-20T10:42:27Z
draft: true
weight: 8
hide: ["header"]

---


This component came to Lucid at a later stage due to the need for grouping common Jobs that together make up one useful functionality, working as a team in a certain sequence to achieve a goal, yet that goal is still one step in the bigger goal (Feature). Technically, *Operation* classes are similar to Features and we would use them the same, meaning that they both have `run($component,$params)` method to run jobs, however conceptually they are completely different. Features can run Operations and Jobs. Operations can run Jobs only.

Their purpose is to increase the degree of code reusability by providing composite functionalities.

Here is an example based on our `CreateArticleFeature` code sample:

Upon creating an article, we would like to send notifications to the subscribers of the author. This functionality is made up of three steps:

1. **Retrieve subscribers**: here we need to be careful for there could be a huge list of them, so we might need to paginate through them and send these notifications in batches
2. **Enqueue notifications jobs**: we sure won't be issuing thousands of notifications at once. Assuming that we are using some external API to perform this action, they must have limitations, such as a maximum amount of "destinations" to send to - say 200?
3. **Return status:** a summary of the number of notifications that were sent and whether it succeeded to enqueue their jobs. This does not mean that the notifications were actually sent, just that they were enqueued to do so, their sending status needs to be followed up depending on the queue (i.e. [Laravel's Horizon](https://laravel.com/docs/master/horizon))

Each of these steps, naturally will have a job to perform them, but they are tightly coupled in the sense that whenever we need to notify subscribers we will need to perform these steps together. If we were to not have an Operation that groups them we would have had to include the three jobs together every time, and this is what Operations helps eliminate.

Here's the snippet:

```php
class NotifySubecribersOperation extends Operation
{
    private $authorId;

    public function __construct(int $authorId)
    {
        $this->authorId = $authorId;
    }

    public function handle()
    {
        $subscribers = $this->run(PaginateSubscribersJob::class, [
            'authorId' => $this->authorId,
        ]);

        $enqueued = $this->run(EnqueueNotificationJob::class, [
            'authorId' => $authorId,
            'subscribers' => $subscribers,
        ]);

        return $this->run(DetermineEnqueueingNotificationJobsStatus::class, [
            'enqueued' => $enqueued,
        ]);
    }
}
```

1. `PaginateSubscribersJob` queries the database based on the given `authorId` and returns an [iterator instance](https://www.php.net/manual/en/class.iterator.php) [taking advantage of [PHP's generators](https://www.php.net/manual/en/language.generators.overview.php) in such case is a great added value in this case].
2. `EnqueueNotificationJob` loops over the iterator instance, determines the means of notifying the subscriber and enqueues the corresponding notification Job.

    Jobs, Operations and Features in Lucid can all be enqueued just like any other Laravel job instance. See how at [LINK TO WORKING WITH QUEUES]

3. `DetermineEnqueueingNotificationJobsStatus` collects the statuses and using some logic determines the status of this operation and returns it back to the calling Feature.

Now we just run that operation within our Feature as we usually do:

```php
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

        $notificationStatus = $this->run(NotifySubecribersOperation::class, [
            'authorId' => Auth::user()->getKey(),
        ]);

        return $this->run(new RespondWithJsonJob($notificationStatus));
    }
}
```


