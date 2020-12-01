---
title: "Domains"
date: 2020-11-08T19:34:03Z
draft: false
weight: 13
hide: ["header"]
---

Inspired by Domain-Driven Development, this piece of the Lucid stack is merely directories that are all about organizing and categorizing
code according to the topic they belong to.

There is no specific way of naming them because it differs per case, but according to our experience over
the years we found that they're usually of two types, the "internal" and the "external" type of domain. Here are a few examples
to illustrate the difference

{{<columns>}}
**Internal**

- Http
    - `RespondWithJsonJob`
    - `RespondWithJsonErrorJob`
    - `RespondWithHtmlJob`
    - `RespondWithHtmlErrorJob`
    - `RespondWithViewJob`
- Chat
    - `SendMessageToChannelJob`

<--->

**External**

- GitHub
    - `LoginWithGitHubJob`
    - `FetchGitHubReposJob`
- Facebook
    - `LoginWithFacebookJob`
    - `FetchUserPostsJob`

{{</columns>}}

Even though they differ in the nature of functionalities they expose, these are still domains from which our application would
(re)use code wherever possible. This separation is only logical to help with the understanding of domains but is not an actual
separation in code.

## Characteristics

- **Isolation**: Domains operate in isolation in the sense that they should never need anything from another domain to accomplish their job.
They may, however, use elements from the framework or Foundation for abstraction reasons and to eliminate code replication.
- **Encapsulation**: Ideally, domains should be structured in a way that would allow them to be moved around with the least amount
of impact on their output. They contain their jobs, requests/validation, tests and other custom classes to get their job done.

Consider domain jobs to be the pieces of code that will be shared and reused the most, they can be called from anywhere
which makes it essential that they do not depend on any other domain.

## What Goes Into A Domain

From the Lucid stack, domains contain Job classes and their tests, acting as the only way of communicating with a domain.
However any other class that belongs to the domain should also be present there for centralisation.

Consider the example of an application that integrates with GitHub to login as a GitHub user and
it allows to fetch a repository's info from GitHub's API:

<pre>
app/Domains/GitHub
├── GitHubClient.php
├── Jobs
│   ├── FetchGitHubRepoInfoJob.php
│   └── LoginWithGitHubJob.php
├── Exceptions
│   ├── InvalidTokenException.php
│   └── RepositoryNotFoundException.php
└── Tests
    └── GitHubClientTest.php
    └── Jobs
        ├── FetchGitHubReposJobTest.php
        └── LoginWithGitHubJobTest.php
</pre>

- `GitHubClient` is the class that the jobs use to communicate with GitHub's API.
- `FetchGitHubRepoInfoJob` contains all the details required to call the API such as URL and query params,
exposing them through parameters in the signature, and using the rest of the classes from the domain:

{{% notice danger %}}
{{<icon name="fa-exclamation-triangle">}}&nbsp;This code snippet is fictional, it is for demonstrative purposes only and is not meant to work as-is in real life.
{{% /notice %}}
```php
<?php

namespace App\Domains\GitHub\Jobs;

use Lucid\Units\Job;
use App\Domains\GitHub\GitHubClient;
use App\Domains\GitHub\Exceptions\InvalidTokenException;
use App\Domains\GitHub\Exceptions\RepositoryNotFoundException;

class FetchGitHubRepoInfoJob extends Job
{
    private $name;
    private $includeCollaborators;

    public function __construct($name, $includeCollaborators = false)
    {
        $this->name = $name;
        $this->includeCollaborators = $includeCollaborators;
    }

    public function handle(GitHubClient $github)
    {
        $params = [];

        if ($this->includeCollaborators) {
            $params = 'with_collaborators';
        }

        $response = $github->repo($this->name, $params);

        switch($response->status) {
            case 404:
                throw new RepositoryNotFoundException();
                break;
            case 403:
                throw new InvalidTokenException();
                break;
        }

        return $response;
    }
}
```

## Testing

Domains contain the tests of their jobs and classes in order to be self-sufficient. They are considered to be unit tests
so that the domain can provide the guarantee of a working unit, that way we focus on feature tests in the rest of the application.

When generating Jobs, their tests are automatically generated in the corresponding locations:

```bash
lucid make:job FetchGitHubRepoInfoJob GitHub
```

Will generate two files:
1. `app/Domains/GitHub/Jobs/FetchGitHubRepoInfoJob.php`
2. `app/Domains/GitHub/Tests/FetchGitHubRepoInfoJobTest.php`

However these tests will not be recognized by PHPUnit, to configure them add the following to `phpunit.xml` under `<testsuites>`
and the next time you run `phpunit` all your tests within domains will be included automatically in the run:

```xml
<testsuite name="Domains">
    <directory suffix="Test.php">./app/Domains</directory>
</testsuite>
```
